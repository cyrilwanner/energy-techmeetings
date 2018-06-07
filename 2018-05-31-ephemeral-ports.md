## Ephemeral Ports

<center><img src="images/ephemeral-ports-flow.png" alt="Ephemeral Ports flow"></center>

When you open a connection to another server, the response is normally not received on the same port but on another one which is currently free and just got opened for receiving an answer.
All systems reserve a specific range for such dynamic ports which are called Ephemeral ports. Most Unix kernels for example use 32768 to 61000.

* they are shortlived ports
* every connection gets another free port so they are not predictable (although they are always in the defined range)

<details>
  <summary>Why do we need those ports?</summary>

  This way, multiple connections to the same destination port can get opened at the same time (e.g. two concurrent website requests) which would not be possible if it would always get returned on the same port
</details>

# Ephemeral Ports and Firewalls

We have defined the following rules in our firewall:

**inbound**:

| Order | Port | Status |
| ----- | ---- | ------ |
| 1 | **22** | ALLOWED |
| 2 | **80** | ALLOWED |
| 3 | **443** | ALLOWED |
| 4 | * | BLOCKED |

**outbound**:

| Order | Port | Status |
| ----- | ---- | ------ |
| 1 | * | ALLOWED |

We obviously want to allow incoming traffic only on ports *22*, *80* and *443* and allow *all* outbound traffic so we can still call 3rd party APIs or open any other connection to external servers.

<details>
  <summary>Will this work or do you see any problems?</summary>

  It depends on whether the firewall is stateless or statefull.

  We had a problem configuring firewalls in our AWS setup recently because we didn't thought about ephemeral ports when we configured the firewalls and so we were able to connect to the instance but any outgoing request on it simply timed out.
</details>

## Stateless vs. stateful firewalls

A stateless firewall..

* treats each packet independently
* is not aware of traffic patterns or data flow

While a stateful firewall..

* can remember connection-level information
* knows if a packet is a response to another request
* knows the current state of a connection

<details>
  <summary>Which firewall will now work as expected with the rules defined above? And how can we change the other one that it also works as expected?</summary>

  The stateful firewall will work as expected as it knows, that even if the incoming port is 32768, it is a response to a connection we have opened and as all outgoing connections should be allowed, the firewall lets the packages pass.

  But the stateless firewall doesn't know that. We have to additionally allow all ports between 32768 and 61000 on the stateless firewall.
</details>

## AWS

<center><img src="images/aws-network-example.png" alt="AWS Network Example"></center>

There are two different kinds of "firewalls" at AWS: Network ACL and Security Groups.

<details>
  <summary>Which one is stateful and which is stateless?</summary>

  The one which is nearer to the EC2 instances is stateful, so Security Groups are stateful while Network ACLs watch the traffic on network level and are stateless.

  So when we want our initial firewall definition to work at AWS, we have to allow the epheremal ports on the Network ACL layer.
</details>
