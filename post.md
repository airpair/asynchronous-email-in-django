One of the most common tasks for a web application is to send email asynchronously. There are multiple ways to send emails asynchrnously in Django. We can send asynchronous email using threads also but using [threads](https://djangosnippets.org/snippets/1982/) is not recommended since it is not scalable. The one we would discuss here will be using [Django Post office](https://github.com/ui/django-post_office), [Celery](https://github.com/pmclanahan/django-celery-email) and [Django Celery Email](https://github.com/celery/django-celery).

## Django Celery

Django celery helps you execute tasks asynchronously  and also supports the scheduling of tasks. Celery requires a message broker to send and receive messages. There are a [wide range of options](http://celery.readthedocs.org/en/latest/getting-started/first-steps-with-celery.html#choosing-a-broker) available for brokers. We'll use [RabbitMQ](http://www.rabbitmq.com/) as the broker here.  You can install celery using:-

```shell
$ pip install django-celery
```
You can install RabbitMQ as follows:

```bash
$ sudo apt-get install rabbitmq-server
```

## Django Post Office

Django post office is a simple application that lets you send and manage emails in django. It has its own email scheduling support and allow you to specify priority with each email. Install Post Office using:-

```bash
$ pip install django-post_office
```

## Django Celery Email

Django celery email uses celery to send asynchronous emails. You can install it using:-

```bash
$ pip install django-celery-email
```
Now once you get all these installed you need to add these applications to installed apps as follows:-

```python
INSTALLED_APPS += (
#other apps
'djcelery',
'djcelery_email',
'post_office'
)
```
Setup database using:-
```bash
$ python manage.py migrate
```
Now you need to add few settings to configure django-celery and post-office. Add  the following to your settings file.
```python
# setup celery
import djcelery
djcelery.setup_loader()

```
Use post office as the default email backend 
```python
EMAIL_BACKEND = 'post_office.EmailBackend'
```
Use djcelery's email backend as a backend for post office 

```python
POST_OFFICE_BACKEND = 'djcelery_email.backends.CeleryEmailBackend'
```
Please note that the default priority for sending emails through post office is Medium . If you want to send emails instantaneously then you need to change the priority to ‘now’.
```python
POST_OFFICE = {
    'DEFAULT_PRIORITY' : 'now'
}
```
Now lets add the mail server settings as follows:-
```python
EMAIL_HOST = 'YOUR_HOST_NAME'
EMAIL_HOST_USER = "YOUR_HOST_USER_NAME"
EMAIL_PORT = 25  # default smtp port
EMAIL_HOST_PASSWORD = "YOUR_HOST_USER_PASSWORD"
EMAIL_USE_TLS = False
DEFAULT_FROM_EMAIL = 'testing@example.com'
```
And lastly you need to start RabbitMQ and  celery server using :-
```bash
$ sudo rabbitmq-server 
$ python manage.py celeryd
```
Now any mails sent using the default send_mail function will be delivered asynchronously using post_office and celery email backend. You can view the list of sent mails in admin panel under Post Office logs. 

There are certain [management commands](https://github.com/ui/django-post_office/blob/master/README.rst#management-commands) available with post office to send the queued mail and delete the email logs.

To send the queued email run:
```bash
$ python manage.py send_queued_mail
```
To delete all emails created before an X number of days (defaults to 90
```bash
$ python manage.py cleanup_mail
```
You can also setup cron jobs to do the same.

This was all about setting up asynchronous emails in Django using Celery and Django Post Office. Feel free to add your comments below.



