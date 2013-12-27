java-messaging-connectivity
===========================

Java Message Connectivity (JMC) is a Java-based messaging access technology. This technology is an API for the Java programming language that defines how a client may access a Messaging Service such as APNS and or GCM. It provides methods for sending messages to a Messaging Service.

Background
----------

Plenty of solutions that interact with Messaging Services such as APNS and GCM have been developed, each with its own implementation. Most of these solutions couple the messaging with pooling of connections to the Messaging Service and none follow a standardized API for ease of use and for decoupling responsabilities.

Objective
---------

The objective is to create a standard JMC API for the community to create their own implementations following a standard. Such standard will allow third party vendors to create Messaging Drivers as well as Messaging Connection Pools in a decoupled architecture.

The concept behind this effort is to follow a very similar standard to JDBC, where JDBC connections are standalone and managed by via a connection pool.

Functionality
-------------

When the JMC driver is loaded, the driver shall register itself to the DriverManager. This can be done by including the needed code in the driver class's static block. E.g., DriverManager.registerDriver(Driver driver)

Now when a connection is needed, one of the DriverManager.getConnection() methods is used to create a JMC connection.

```
Connection conn = DriverManager.getConnection("jmc:somejmcvendor:other data needed by some jdbc vendor", props);
try {
     /* you use the connection here */
} finally {
    //It's important to close the connection when you are done with it
    try {
      conn.close();
    } catch (Throwable ignore) {
      /* Propagate the original exception instead of this one that you may want just logged */
    }
}
```

The URL used is dependent upon the particular JMC driver. It will always begin with the "jmc:" protocol, but the rest is up to the particular vendor. Once a connection is established, a message can be sent.

Since every Messaging Service has a different Message Construct, a simple Message object is used:

```
Message message = new Message();
StringBuffer vendorMessage = new StringBuffer();
// append vendor specific content
message.setMessage(vendorMessage.toString());
```

The message can then be sent to the connection as follows:

```
conn.sendMessage(message);
```

If a message operation fails, JMC raises an MessageException. There is typically very little one can do to recover from such an error, apart from logging it with as much detail as possible.

```
try {
  conn.sendMessage(message);
} catch(MessageException e) {
  e.prinStackTrace();
}
```

JMC Driver
----------

A JMC driver is a software component enabling a Java application to interact with a Messaging Service.

To connect with individual Messaging Services, JMC (the Java Messaging Connectivity API) requires drivers for each Messaging Service. The JMC driver gives out the connection to the Messaging Service and implements the protocol for transferring the message and result between client and Messaging Service.

JMC Connection Pools
--------------------

The JMC API decouples responsabilities allowing third parties to develop connection pools. Such connection pools should be done to enhance the performance of sending messages to the Messaging Services. Opening and maintaining a connection for each Messaging Service, is costly and wastes resources.
