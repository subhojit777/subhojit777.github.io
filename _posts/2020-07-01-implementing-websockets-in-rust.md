---
title: Implementing WebSockets in Rust
description: Implementing WebSockets in Rust
date: '2020-07-01'
author: Subhojit Paul
#tags:
# - "rust language"
# - "actix"
# - "websocket"
# - "webservice"
---

Recently, I needed WebSockets in my side project. I will share how I implemented it, what I learned.

## A little about my side project

The [Questionnaire](https://github.com/subhojit777/questionnaire-backend) improves the experience of the audience during a session. Consider, there is a session going on about "Everything about Demographics"; the presenter asked a question to the audience that "In which continent do you live?" Now, the people will raise hands based on the name mentioned by the presenter one-by-one. A person will not be able to know how many people in the audience belong to continent Asia. The Questionnaire will show the answers in real-time on the presenter screen. It helps to make the session more engaging.

In the Questionnaire, there are two screens - Presenter and Participant. The Presenter screen is where the presenter is showing the slides, and the presenter will show the question there. And, it can be answered from the Participant screen. The audience can access the Participant Screen from their smartphones, basically in a Browser.

The backend of the Questionnaire is in Rust. It exposes web services. The frontend - a React application consumes those services.

## Where WebSocket comes into the picture

I needed a mechanism so that:

1. The audience sees the options of the same question that the presenter has asked - they are in sync
2. Show answers real-time on the presenter screen (this feature is still in progress)

Please note that I will be using the "presenter + audience sync" feature to demonstrate the implementation of WebSocket in this article.

## Implementing the WebSocket

The application uses [`actix-web`](https://crates.io/crates/actix-web) for the WebServices. I have used [`actix-broker`](https://crates.io/crates/actix-broker) for the implementation of the WebSocket.

`actix-broker` facilitates running of the WebSocket in the same thread as the HttpServer. Spawning a new thread for every Presenter Screen, and keeping them in sync was difficult. That is why I chose to run the WebSocket in the same thread. The handling of the WebSocket request and response is asynchronous.

### The WebSocket endpoint

```rust
use actix_web::HttpRequest;
use actix_web::web::{Data, Payload};
use actix_web::{get, HttpResponse};
use actix_web_actors::ws;

#[get("/ws/")]
pub async fn index(
    request: HttpRequest,
    stream: Payload,
    pool: Data<Pool<ConnectionManager<MysqlConnection>>>,
) -> Result<HttpResponse, actix_web::Error> {
    let connection = pool.get().expect("unable to get database connection");
    let response = ws::start(WebSocketSession::new(connection), &request, stream);
    response
}
```

The above is the implementation of the WebSocket endpoint. The client (the React app) tries to connect with the WebSocket via `/ws/` endpoint.

```js
const client = new w3cwebsocket('ws://127.0.0.1:8088/ws/'); // 127.0.0.1:8088 is the URL of the Rust HTTPServer.
client.onopen = () => console.log("websocket client connected.");
```

During the connection, the application creates a new WebSocket session. There will be multiple sessions for everyone in the audience, including the presenter. It also requires data to be fetched from a database, and therefore, I am passing the database connection as well.

Behind the scenes, the WebSocket session is an [`actor`](https://actix.rs/docs/whatis). Next, I will show you how it is handled.

### Handling of sessions

The following code snippet is taken from the side-project.

```rust
use actix::prelude::*;
use actix_web_actors::ws::WebsocketContext;
use rand::prelude::ThreadRng;
use actix_broker::BrokerSubscribe;

impl Actor for WebSocketSession {
    type Context = WebsocketContext<Self>;

    fn started(&mut self, ctx: &mut Self::Context) {
        let join_session = JoinSession(ctx.address().recipient());
        WebSocketServer::from_registry()
            .send(join_session)
            .into_actor(self)
            .then(|id, act, _ctx| {
                if let Ok(id) = id {
                    act.id = id;
                }

                fut::ready(())
            })
            .wait(ctx);
    }
}

#[derive(Clone, Message)]
#[rtype(result = "usize")]
pub struct JoinSession(pub Recipient<Message>);

#[derive(Message)]
#[rtype(result = "()")]
pub struct Message(pub String);

#[derive(Default)]
pub struct WebSocketServer {
    sessions: HashMap<usize, Recipient<Message>>,
    rng: ThreadRng,
}

impl WebSocketServer {
    pub fn add_session(&mut self, client: Recipient<Message>) -> usize {
        let id: usize = self.rng.gen();

        self.sessions.insert(id, client);

        id
    }
}

impl Actor for WebSocketServer {
    type Context = Context<Self>;

    fn started(&mut self, ctx: &mut Self::Context) {
        self.subscribe_system_async::<SendMessage>(ctx);
    }
}

impl Handler<JoinSession> for WebSocketServer {
    type Result = MessageResult<JoinSession>;

    fn handle(&mut self, msg: JoinSession, _ctx: &mut Self::Context) -> Self::Result {
        let JoinSession(client) = msg;

        let id = self.add_session(client);

        MessageResult(id)
    }
}
```

`impl Actor for WebSocketSession {}` - This defines the behaviour of WebSocket session. When the session starts, it triggers the `JoinSession`. `WebSocketServer` is another Actor which keeps track of the sessions. You will notice `self.subscribe_system_async::<SendMessage>(ctx);` inside the implementation of the server - I will come to it later.

The new session is pushed to a `HashMap` when `JoinSession` is triggered.

### Making sure presenter and audience are in sync

The following code demonstrates how I am making sure that the audience sees the options of the same question that the presenter has asked.

```rust
use actix_broker::BrokerIssue;
use actix_broker::BrokerSubscribe;

impl Actor for WebSocketServer {
    type Context = Context<Self>;
    fn started(&mut self, ctx: &mut Self::Context) {
        self.subscribe_system_async::<SendMessage>(ctx);
    }
}

impl WebSocketServer {
    pub fn send_message(&mut self, message: String) {
        for (_id, recipient) in &self.sessions {
            recipient
                .do_send(Message(message.to_owned()))
                .expect("Could not send message to the client.");
        }
    }
}

impl Handler<SendMessage> for WebSocketServer {
    type Result = ();
    fn handle(&mut self, msg: SendMessage, _ctx: &mut Self::Context) {
        self.send_message(msg.content);
    }
}

impl WebSocketSession {
    pub fn send_msg(&self, msg: String) {
        let msg = SendMessage {
            name: self.name.clone(),
            id: self.id,
            content: msg,
        };

        self.issue_system_async(msg);
    }
}

impl StreamHandler<Result<ws::Message, ProtocolError>> for WebSocketSession {
    fn handle(&mut self, item: Result<ws::Message, ProtocolError>, ctx: &mut Self::Context) {
        match item {
            Ok(Ping(_msg)) => { /* Skipped for simplifcation. */ }
            Ok(Pong(_msg)) => { /* Skipped for simplifcation. */ }
            Ok(Text(text)) => {
                // Presenter (the client) wants to show the next question.
                // Client has sent the request to navigate.
                // Retrieve the next question's data from database, and notify the audience sessions
                // to shift to the next question's options.
                let response = WebSocketResponse { new_question_index };
                self.send_msg(serde_json::to_string(&response).expect("Could not parse to JSON."));
            }
            _ => ctx.stop(),
        }
    }
}
```

The application is listening to any messages from a session - in this case, the presenter's session. For simplification, I have skipped the gory details about how the next question's data is retrieved. Notice that, it is just calling `WebSocketSession`'s `send_msg`.

Using `actix-broker`, the `WebSocketServer` has subscribed to the `SendMessage` - which is responsible for making sure that every Participant Screens show the next question's options.

Inside `send_message`, I am looping through every active session, and sending the response to update the screens.

### Handle cleanups

A person from the audience may wish to no longer participate in the session and closes one's screen. The application needs to make sure that it does not sends the response to that closed session.

```rust
#[derive(Clone, Message)]
#[rtype(result = "()")]
pub struct RemoveSession(pub usize);

impl Actor for WebSocketSession {
    type Context = WebsocketContext<Self>;
    fn stopped(&mut self, _ctx: &mut Self::Context) {
        let remove_session = RemoveSession(self.id);
        self.issue_system_async(remove_session);
    }
}

impl Actor for WebSocketServer {
    type Context = Context<Self>;
    fn started(&mut self, ctx: &mut Self::Context) {
        self.subscribe_system_async::<RemoveSession>(ctx);
    }
}

impl WebSocketServer {
    pub fn remove_session(&mut self, session_id: usize) {
        self.sessions.remove(&session_id);
    }
}

impl Handler<RemoveSession> for WebSocketServer {
    type Result = MessageResult<RemoveSession>;
    fn handle(&mut self, msg: RemoveSession, _ctx: &mut Self::Context) -> Self::Result {
        let RemoveSession(id) = msg;
        self.remove_session(id);
        MessageResult(())
    }
}
```

The application is aware of any session being closed. Same as `started`, the `stopped` handles the closing of a `WebSocketSession`. There, I am issuing a `RemoveSession` event, which in turn removes the session `id` from the `HashMap`.

## Closing Note

I got a lot of help from the [actix examples](https://github.com/actix/examples) - [websocket-chat](https://github.com/actix/examples/tree/master/websocket-chat) and [websocket-chat-broker](https://github.com/actix/examples/tree/master/websocket-chat-broker). Initially, I was following `websocket-chat`, and I realised that it runs the WebSocket in a separate thread. Keeping the sessions in sync was difficult. That is when I referred `websocket-chat-broker`.

The [Rust Debugger](https://marketplace.visualstudio.com/items?itemName=vadimcn.vscode-lldb) in VSCode was super useful. I was not cleaning up the sessions initially, and it resulted in WebSocket crashes. I was able to find out that the WebSocket server is trying to send data to an invalid WebSocket session, and that is why it is crashing.

## Credits

Thanks to [Lalit Shandilya](https://www.linkedin.com/in/shanlalit) for his idea about this project.

Thanks to [Bassam Ismail](https://www.linkedin.com/in/skippednote) for his suggestion about using WebSockets for keeping the presenter and audience screens in sync.
