=jsonrpc4j - JSON-RPC for Java=
This project aims to provide the facility to easily implement JSON-RPC for the java programming language.  jsonrpc4j uses the [http://jackson.codehaus.org/ Jackson] library to convert java objects to and from json objects (and other things related to JSON-RPC). 

*Features Include:*
  * Streaming server ({{{InputStream}}} \ {{{OutputStream}}})
  * HTTP Server ({{{HttpServletRequest}}} \ {{{HttpServletResponse}}})
  * Portlet Server ({{{ResourceRequest}}} \ {{{ResourceResponse}}})
  * Socket Server ({{{StreamServer}}})
  * Integration with the Spring Framework ({{{RemoteExporter}}})
  * Streaming client
  * HTTP client
  * Dynamic client proxies
  * Annotations support
  * Custom error resolving
  * Composite services

==Download==
|| *Name* || *Link* ||
|| Stable || [http://jsonrpc4j.googlecode.com/svn/maven/repo/com/googlecode/jsonrpc4j/0.28/ jsonrpc4j-0.28] ||
|| Development || [http://jsonrpc4j.googlecode.com/svn/maven/repo/com/googlecode/jsonrpc4j/0.29-SNAPSHOT/ jsonrpc4j-0.29-SNAPSHOT] ||
|| Older || [http://jsonrpc4j.googlecode.com/svn/maven/repo/com/googlecode/jsonrpc4j/ older] ||
|| || [ChangeLog Change Log] ||

<g:plusone size="medium"></g:plusone>

<wiki:toc />

=Maven=
This project is built with [http://maven.apache.org Maven]. Be sure to check the pom.xml for the dependencies if you're not using maven.  If you're already using spring you should have most (if not all) of the dependencies already - outside of maybe the [http://jackson.codehaus.org/ Jackson library].  The jsonrpc4j maven repository is located at http://jsonrpc4j.googlecode.com/svn/maven/repo/.  Add the following to your pom.xml if you're using maven:

In {{{<repositories>}}}:
{{{
    <repository>
        <id>jsonrpc4j-webdav-maven-repo</id>
        <name>jsonrpc4j maven repository</name>
        <url>http://jsonrpc4j.googlecode.com/svn/maven/repo/</url>
        <layout>default</layout>
    </repository>
}}}

In {{{<dependencies>}}}:
{{{
	<dependency>
		<groupId>com.googlecode</groupId>
		<artifactId>jsonrpc4j</artifactId>
		<version>0.28</version>
	</dependency>
	<dependency>
		<groupId>com.fasterxml.jackson.core</groupId>
		<artifactId>jackson-core</artifactId>
		<version>2.0.2</version>
	</dependency>
	<dependency>
		<groupId>com.fasterxml.jackson.core</groupId>
		<artifactId>jackson-databind</artifactId>
		<version>2.0.2</version>
 	</dependency>
 	<!-- optional, add if using jackon's annotations -->
	<dependency>
		<groupId>com.fasterxml.jackson.core</groupId>
		<artifactId>jackson-annotations</artifactId>
		<version>2.0.2</version>
	</dependency>
}}}

=JSON-RPC specification=
There doesn't seem to be an official source for the JSON-RPC specification.  With that said, the guys over at [http://groups.google.com/group/json-rpc json-rpc google group] seem to be fairly active so the specification that they've outlined is what was used.

=Streaming server and client=
Jsonrpc4j comes with a streaming server and client to support applications of all types (not just HTTP).  The {{{JsonRpcClient}}} and {{{JsonRpcServer}}} have simple methods that take {{{InputStream}}}s and {{{OutputStream}}}s.  Also in the library is a {{{JsonRpcHttpClient}}} which extends the {{{JsonRpcClient}}} to add HTTP support.

=Spring Framework=
jsonrpc4j provides a {{{RemoteExporter}}} to expose java services as JSON-RPC over HTTP without requiring any additional work on the part of the programmer.  The following example explains how to use the {{{JsonServiceExporter}}} within the Spring Framework.

Create your service interface:
{{{
package com.mycompany;
public interface UserService {
    User createUser(String userName, String firstName, String password);
    User createUser(String userName, String password);
    User findUserByUserName(String userName);
    int getUserCount();
}
}}}

Implement it:
{{{
package com.mycompany;
public class UserServiceImpl
    implements UserService {

    public User createUser(String userName, String firstName, String password) {
        User user = new User();
        user.setUserName(userName);
        user.setFirstName(firstName);
        user.setPassword(password);
        database.saveUser(user)
        return user;
    }

    public User createUser(String userName, String password) {
        return this.createUser(userName, null, password);
    }

    public User findUserByUserName(String userName) {
        return database.findUserByUserName(userName);
    }

    public int getUserCount() {
        return database.getUserCount();
    }

}
}}}

Configure your service in spring as you would any other RemoteExporter:
{{{
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
		http://www.springframework.org/schema/beans/spring-beans-2.5.xsd">

    <bean class="org.springframework.web.servlet.handler.BeanNameUrlHandlerMapping"/>

    <bean id="userService" class="com.mycompany.UserServiceImpl">
    </bean>

    <bean name="/UserService.json" class="com.googlecode.jsonrpc4j.spring.JsonServiceExporter">
        <property name="service" ref="userService"/>
        <property name="serviceInterface" value="com.mycompany.UserService"/>
    </bean>

</beans>
}}}

Your service is now available at the URL /UserService.json.  Type conversion of JSON->Java and Java->JSON will happen for you automatically.  This service can be accessed by any JSON-RPC capable client, including the {{{JsonProxyFactoryBean}}}, {{{JsonRpcClient}}} and {{{JsonRpcHttpClient}}} provided by this project:

{{{
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean  class="com.googlecode.jsonrpc4j.spring.JsonProxyFactoryBean">
        <property name="serviceUrl" value="http://example.com/UserService.json"/>
        <property name="serviceInterface" value="com.mycompany.UserService"/>
    </bean>

<beans>
}}}

In the case that your JSON-RPC requires named based parameters rather than indexed parameters an annotation can be added to your service interface (this also works on the service implementation for the ServiceExporter):
{{{
package com.mycompany;
public interface UserService {
    User createUser(@JsonRpcParamName("theUserName") String userName, @JsonRpcParamName("thePassword") String password);
}
}}}

By default all error message responses contain the the message as returned by Exception.getmessage() with a code of 0.  This is not always desirable.  jsonrpc4j supports annotated based customization of these error messages and codes, for example:

{{{
package com.mycompany;
public interface UserService {
    @JsonRpcErrors({
        @JsonRpcError(exception=UserExistsException.class,
            code=-5678, message="User already exists", data="The Data"),
        @JsonRpcError(exception=Throwable.class,code=-187)
    })
    User createUser(@JsonRpcParamName("theUserName") String userName, @JsonRpcParamName("thePassword") String password);
}
}}}

The previous example will return the error code {{{-5678}}} with the message {{{User already exists}}} if the service throws a UserExistsException.  In the case of any other exception the code {{{-187}}} is returned with the value of {{{getMessage()}}} as returned by the exception itself.

==Auto Discovery With Annotations==
Spring can also be configured to auto-discover services and clients with annotations.

To configure auto-discovery of annotated services first annotate the service interface:
{{{
@JsonRpcService("/path/to/MyService")
interface MyService {
... service methods ...
}
}}}

and use the following configuration to allow spring to find it:
{{{
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean class="com.googlecode.jsonrpc4j.spring.AutoJsonRpcServiceExporter"/>

  <bean class="com.mycompany.MyServiceImpl" />

</beans>
}}}

Configuring a client is just as easy:
{{{
<beans xmlns="http://www.springframework.org/schema/beans"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">

  <bean class="com.googlecode.jsonrpc4j.spring.AutoJsonRpcClientProxyCreator">
    <property name="baseUrl" value="http://hostname/api/" />
    <property name="scanPackage" value="com.mycompany.services" />
  </bean>

</beans>
}}}
Where the {{{baseUrl}}} is added to the front of the path value provided by the {{{JsonRpcService}}} annotation and {{{scanPackage}}} tells spring which packages to scan for services.

=Without the Spring Framework=
jsonrpc4j can be used without the spring framework as well.  In fact, the client and server both work in an Android environment.

==Client==
Here's an example of how to use the client to communicate with the JSON-RPC service described above:

{{{
JsonRpcHttpClient client = new JsonRpcHttpClient(
    new URL("http://example.com/UserService.json"));

User user = client.invoke("createUser", new Object[] { "bob", "the builder" }, User.class);
}}}

Or, the ProxyUtil class can be used in conjunction with the interface to create a dynamic proxy:
{{{
JsonRpcHttpClient client = new JsonRpcHttpClient(
    new URL("http://example.com/UserService.json"));

UserService userService = ProxyUtil.createClientProxy(
    getClass().getClassLoader(),
    UserService.class,
    client);

User user = userService.createUser("bob", "the builder");
}}}

==Server==
The server can be used without spring as well:

{{{
// create it
JsonRpcServer server = new JsonRpcServer(userService, UserService.class);
}}}

After having created the server it's simply a matter of calling one of the {{{handle(...)}}} methods available.  For example, here's a servlet using the very same {{{UserService}}}:

{{{
class UserServiceServlet 
    extends HttpServlet {

    private UserService userService;
    private JsonRpcServer jsonRpcServer;

    protected void doPost(HttpServletRequest req, HttpServletResponse resp) {
        jsonRpcServer.handle(req, resp);
    }

    public void init(ServletConfig config) {
        this.userService = ...;
        this.jsonRpcServer = new JsonRpcServer(this.userService, UserService.class);
    }

}
}}}

===Composite Services===
Multiple services can be combined into a single server using one of the {{{ProxyUtil::createCompositeService(...)}}} methods.  For example:

{{{
UserverService userService = ...;
ContentService contentService = ...;
BlackJackService blackJackService = ...;

Object compositeService = ProxyUtil.createCompositeServiceProxy(
    this.getClass().getClassLoader(),
    new Object[] { userService, contentService, blackJackService},
    new Class<?>[] { UserService.class, ContentService.class, BlackJackService.class},
    true);

// now compositeService can be used as any of the above service, ie:
User user = ((UserverService)compositService).createUser(...);
Content content =  ((ContentService)compositService).getContent(...);
Hand hand = ((BlackJackService)compositService).dealHand(...);

}}}

This can be used in conjunction with the {{{JsonRpcServer}}} to expose the service methods from all services at a single location:

{{{
JsonRpcServer jsonRpcServer = new JsonRpcServer(compositeService);
}}}

A spring service exporter exists for creating composite services as well named {{{CompositeJsonServiceExporter}}}.

===Streaming (Socket) Server===
A streaming server that uses {{{Socket}}}s is available in the form of the {{{StreamServer}}} class.  It's use is very straitforward:

{{{
// create the jsonRpcServer
JsonRpcServer jsonRpcServer = new JsonRpcServer(...);

// create the stream server
int maxThreads = 50;
int port = 1420;
InetAddress bindAddress = InetAddress.getByName("...");
StreamServer streamServer = new StreamServer(
    jsonRpcServer, maxThreads, port, bindAddress);

// start it, this method doesn't block
streamServer.start();
}}}

and when you're ready to shut the server down:

{{{
// stop it, this method blocks until
// shutdown is complete
streamServer.stop();
}}}

Of course, this is all possible in the Spring Framework as well:

{{{
    <bean id="streamingCompositeService" class="com.googlecode.jsonrpc4j.spring.CompositeJsonStreamServiceExporter">
        <!-- can be an IP, hostname or omitted to listen on all available devices -->
        <property name="hostName" value="localhost"/>
        <property name="port" value="6420"/>
        <property name="services">
        	<list>
	        	<ref bean="userService" 	/>
	        	<ref bean="contentServic" 	/>
	        	<ref bean="blackJackService" 	/>
        	</list>
        </property>
    </bean>
}}}

==={{{JsonRpcServer}}} settings explained===
The following settings apply to both the {{{JsonRpcServer}}} and {{{JsonServiceExporter}}}:

  * *_allowLessParams_* - Boolean specifying whether or not the server should allow for methods to be invoked by clients supplying less than the required number of parameters to the method.
  * *_allowExtraParams_* - Boolean specifying whether or not the server should allow for methods to be invoked by clients supplying more than the required number of parameters to the method.
  * *_rethrowExceptions_* - Boolean specifying whether or not the server should re-throw exceptions after sending them back to the client.
  * *_backwardsComaptible_* - Boolean specifying whether or not the server should allow for jsonrpc 1.0 calls.  This only includes the omission of the jsonrpc property of the request object, it will not enable class hinting.
  * *_errorResolver_* - An implementation of the {{{ErrorResolver}}} interface that resolves exception thrown by services into meaningful responses to be sent to clients.  Multiple {{{ErrorResolver}}}s can be configured using the {{{MultipleErrorResolver}}} implementation of this interface.

===Server Method resolution===
Methods are resolved in the following way, each step immediately short circuits the process when the available methods is 1 or less.

  # All methods with the same name as the request method are considered
  # If *_allowLessParams_* is disabled methods with more parameters than the request are removed
  # If *_allowExtraParams_* is disabled then methods with less parameters than the request are removed
  # If either of the two parameters above are enabled then methods with the lowest difference in parameter count from the request are kept
  # Parameters types are compared to the request parameters and the method(s) with the highest number of matching parameters is kept
  # If there are multiple methods remaining then the first of them are used

jsonrpc4j's method resolution allows for overloaded methods _sometimes_.  Primitives are easily resolved from json to java.  But resolution between other objects are not possible.


For example, the following overloaded methods will work just fine:

json request:
{{{
{jsonrpc:"2.0", id:10, method:"aMethod", params:["Test"]}
}}}

java methods:
{{{
public void aMethod(String param1);
public void aMethod(String param1, String param2);
public void aMethod(String param1, String param2, int param3);
}}}

But the following will not:

json request:
{{{
{jsonrpc:"2.0", id:10, method:"addFriend", params:[{username:"example", firstName:"John"}]}
}}}

java methods:
{{{
public void addFriend(UserObject userObject);
public void addFriend(UserObjectEx userObjectEx);
}}}

The reason being that there is no efficient way for the server to determine the difference in the json between the {{{UserObject}}} and {{{UserObjectEx}}} pojos.
