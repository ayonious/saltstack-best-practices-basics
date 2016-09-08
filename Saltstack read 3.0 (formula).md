###what is ZeroMQ? what does it have to do with salt?

Ayons ans: ZeroMQ is somekind of way of sending info from master to minions.
I think it is eqivalent to ssh but much faster.


Read from: http://stackoverflow.com/questions/4499510/what-are-zeromq-use-cases?rq=1

It's a distributed messaging product. ZeroMQ is to standard messaging brokers what git is to svn. Good introductory talk by Ian Barber: http://vimeo.com/20605470

ZeroMQ, as it's name suggests most probably is a messaging provider. A messaging API is needed to send and recieve messages using these message providers. And you need to integrate these providers with your application server (view documentation). Some MQ supports multiple platforms like Ruby, Java, Php and Others. It is used for loose coupling between two modules in an enterprise application. If you are a Java Programmer, refer to JMS Specifications (Java Mesaging Service) at Oracle's site.




You can directly import formulas from git hubs 

https://github.com/saltstack-formulas


