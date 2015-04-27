---
layout: post
title: Unable to send e-mail. Contact the site administrator if the problem persists.
date: '2012-09-24T15:35:00.003+05:30'
author: Subhojit Paul
tags:
- drupal_mail()
- drupal
- Ubuntu
- mail()
modified_time: '2013-04-23T11:42:47.860+05:30'
thumbnail: http://1.bp.blogspot.com/-7qGtS0-KXok/T_CEiz2uoyI/AAAAAAAAABY/yiVY5pKU4xc/s72-c/Screenshot-1.png
blogger_id: tag:blogger.com,1999:blog-6340784984471653280.post-7018136598191482441
blogger_orig_url: http://subhojitpaul.blogspot.com/2012/09/unable-to-send-e-mail-contact-site.html
---

Drupal newbies trying to send mails from the Drupal contact form or using [drupal_mail()](http://api.drupal.org/api/drupal/includes!mail.inc/function/drupal_mail), usually gets this error message “Unable to send e-mail. Contact the site administrator if the problem persists.”. This is because your system is not configured to send emails.
[drupal_mail()](http://api.drupal.org/api/drupal/includes!mail.inc/function/drupal_mail) calls the PHP [mail()](http://php.net/manual/en/function.mail.php) function to send emails, hence this article can also be useful to those PHP programmers who do not use Drupal as CMS.
To send e-mail, you need to install and configure postfix in your Ubuntu machine.

**What is postfix?**

> Postfix is the default Mail Transfer Agent (MTA) in Ubuntu. It attempts to be fast and easy to administer and secure. It is compatible with the MTA sendmail.

For further reading see this [page](https://help.ubuntu.com/10.04/serverguide/postfix.html)

To install postfix enter the following codes:

```bash
sudo apt-get install php-pear
sudo pear install mail
sudo pear install Net_SMTP
sudo pear install Auth_SASL
sudo pear install mail_mime
sudo apt-get install postfix
```

This will install required packages and postfix in you Ubuntu machine.
For users who have postfix already installed in their system will get a  message that "postfix is already installed".
Configure postfix using the following command:
```bash
sudo dpkg-reconfigure postfix
```

After you enter the command you will be asked certain questions. Most of the answers will be their default answers, but since you may be new to networking I have also specified the answers with their explanations.

*   The first question will ask you to enter mail server configuration type. The options with their details are listed there. Choose “Internet Site” option since you would like to send and recieve mails using SMTP.

    [![](http://1.bp.blogspot.com/-7qGtS0-KXok/T_CEiz2uoyI/AAAAAAAAABY/yiVY5pKU4xc/s320/Screenshot-1.png)](http://1.bp.blogspot.com/-7qGtS0-KXok/T_CEiz2uoyI/AAAAAAAAABY/yiVY5pKU4xc/s1600/Screenshot-1.png)

*   The next question asks you to specify mail name. The default will be the username. Do not change it and move to the next question.

    [![](http://3.bp.blogspot.com/-IcRAbG2L59M/UGA2gCFRPEI/AAAAAAAAADg/FRIUznkcFf8/s320/Screenshot-2.png)](http://3.bp.blogspot.com/-IcRAbG2L59M/UGA2gCFRPEI/AAAAAAAAADg/FRIUznkcFf8/s1600/Screenshot-2.png)

*   Next question asks you to enter the name by which you will recieve mails. The details are listed there. You will want that mails sent to you are saved in /var/mail/<your-username>, so do not change the default answer i.e. your username.

    [![](http://4.bp.blogspot.com/-OVy2ZiTWRNw/UGAv1K7APbI/AAAAAAAAAC4/QksE4fyZuOo/s320/Screenshot-3.png)](http://4.bp.blogspot.com/-OVy2ZiTWRNw/UGAv1K7APbI/AAAAAAAAAC4/QksE4fyZuOo/s1600/Screenshot-3.png)


*   The next question asks you to enter the IP address from where you can accept mails. The defaults will be your username and localhost. Since, you will sending mails from you localhost Drupal site so do not change default answer and move to the next question.

    [![](http://2.bp.blogspot.com/-lCgl3MV_Wis/T_CEypGHPEI/AAAAAAAAABw/A0QNGosHNBc/s320/Screenshot-4.png)](http://2.bp.blogspot.com/-lCgl3MV_Wis/T_CEypGHPEI/AAAAAAAAABw/A0QNGosHNBc/s1600/Screenshot-4.png)

*   In the next question choose the default answer “no”.

    [![](http://1.bp.blogspot.com/-SSrBqixEGVk/T_CEznOORtI/AAAAAAAAAB0/yYJCfVyQRS0/s320/Screenshot-5.png)](http://1.bp.blogspot.com/-SSrBqixEGVk/T_CEznOORtI/AAAAAAAAAB0/yYJCfVyQRS0/s1600/Screenshot-5.png)

*   In the next question you are asked to enter the IP addresses which you want to block. Default answer are your localhost IP address i.e 127.0.0.1\. So, you have to clear this field.

    [![](http://1.bp.blogspot.com/-Uid0kqysOfQ/T_CE0lD0qKI/AAAAAAAAACA/ISYX0UgLAgo/s320/Screenshot-6.png)](http://1.bp.blogspot.com/-Uid0kqysOfQ/T_CE0lD0qKI/AAAAAAAAACA/ISYX0UgLAgo/s1600/Screenshot-6.png)

*   Select the default answer 0 in mailbox size limit.

    [![](http://4.bp.blogspot.com/-pvxSVk82Dtc/T_CE2QrIJJI/AAAAAAAAACI/0IjnPIBH-S0/s320/Screenshot-7.png)](http://4.bp.blogspot.com/-pvxSVk82Dtc/T_CE2QrIJJI/AAAAAAAAACI/0IjnPIBH-S0/s1600/Screenshot-7.png)

*   Select the default answer in next question and do not make any changes in the field.

    [![](http://2.bp.blogspot.com/-Y8X-Fvc4bhc/T_CE3z7_g4I/AAAAAAAAACQ/9NKOg8HexCQ/s320/Screenshot-8.png)](http://2.bp.blogspot.com/-Y8X-Fvc4bhc/T_CE3z7_g4I/AAAAAAAAACQ/9NKOg8HexCQ/s1600/Screenshot-8.png)

*   Select “all” in next question.

    [![](http://1.bp.blogspot.com/-eswIR6xFKts/T_CE5EFm9TI/AAAAAAAAACY/RqfVLSPI-Fs/s320/Screenshot-9.png)](http://1.bp.blogspot.com/-eswIR6xFKts/T_CE5EFm9TI/AAAAAAAAACY/RqfVLSPI-Fs/s1600/Screenshot-9.png)

Postfix in your Ubuntu machine is now configured and now you can send mail from Drupal's contact form or using [drupal_mail()](http://api.drupal.org/api/drupal/includes!mail.inc/function/drupal_mail).
