# Riak Security

## Riak

Riak is a powerful open-source distributed database focused on scaling predictably and easily, while remaining highly available in the face of server crashes, network partitions or other (inevitable) disasters.

## Commitment

Data security is an important and sensitive issue to many of our users. A real-world approach to security allows us to balance appropriate levels of security and related overhead while creating a fast, scalable, and operationally straightforward database.

*An executive-level quote about our commitment to security would be good here*

### Continuous Improvement

Though we make every effort to thwart security vulnerabilities whenever possible (including through independent reviews), no system is completely secure. We will never claim that Riak is 100% secure (and you should seriously doubt anyone who claims their solution is). What we can promise is that we openly accept all vulnerabilities from the community. When appropriate, we'll publish and make every attempt to quickly address these concerns.

### Balance

More layers of security increase operational and administrative costs. Sometimes those costs are warranted, sometimes they are not. Our approach is to strike an appropriate balance between effort, cost and security.

For example, Riak does not have fine-grained role-base security. Though it can be an attractive bullet-point in a database comparison chart, you're usually better off finely controlling data access through your application or a service layer.

### Notifying Basho 

If you discover a potential security issue, please email us at security@basho.com, and allow us 48 hours to reply.

We prefer to be contacted first, rather than searching for blog posts over the internet. This allows us to open a
dialog with the security community on how best to handle a possible exploit without putting any users at risk.

For sensative topics, you may send a secure message. The security team's GPG key is:

*GPG HERE*

## Security Best Practices

### Network Configurations

Being a distributed database means that much of its security springs from how you configure your network. We have a few recommendations for [Network Security and Firewall Configurations](/Network-Security-and-Firewall-Configurations.html).

### Client Auth

Many of the Riak drivers support HTTP basic auth, though this is not a roll-based security. You might instead wish to connect over HTTPS or through a VPN.

### Multi Data Center Replication

For those version of Riak that support Multi Data Center (MDC) Replication, you can configure Riak 1.2+ to communicate over SSL, to seemlessly encrypt the message traffic.

*No link here yet until the EDS docs are published*