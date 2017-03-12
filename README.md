In this article, we will explore some of the benefits of dependency injection and how this can be used in all types of programming, including small and simple programs.  Some of the example shown below make use of Guice (<a href="http://code.google.com/p/google-guice/" target="_blank">Homepage</a>), a dependency injection framework by Google.  The same concept applies for any other dependency injection framework such as Spring (<a href="http://spring.io/" target="_blank">Homepage</a>).

Please note that this article does not provide detailed description of Guice or any other dependency injection framework, but it only focuses on the benefits of dependency injection and design methods that improve modularity, extendibility and testing.  The main idea behind this article is to understand why dependency injection should be used rather than how to implement it.

All code listed below is available at: <a href="https://github.com/javacreed/why-should-we-use-dependency-injection" target="_blank">https://github.com/javacreed/why-should-we-use-dependency-injection</a>.  Most of the examples will not contain the whole code and may omit fragments which are not relevant to the example being discussed.  The readers can download or view all code from the above link.

<h2>What is dependency injection?</h2>

Let say we want to make some tea.  How would we do it and what do we need?  We need some boiled water, a clean mug, teabags, sugar and milk (at least that's what I put into mine).  In order to make tea we need to provide all these ingredients and we need to manage all these ourselves.  We have to fill the kettle with water and heat it up, get the milk from the refrigerator and get the rest. 

Now assume that we can make tea, by simply asking someone else to do it for us.  Someone that knows how to do the tea just the way we want it.  Would that be better than making the tea ourselves?  If yes, then welcome to dependency injection. 

Dependency injection is a framework that takes care of creating objects for us without having to worry about providing the right <em>ingredients</em> so to say.

<h2>A Simple Example</h2>

Let say we have a class, <code>Person</code>, and this class needs to send a message.  The <code>Person</code> class requires the aid of some other class, <code>Email</code>, in order to send a message.  Following is a simple way of doing this.

<pre>
package com.javacreed.examples.di.part1;

public class Email {
 public void sendEmail(String subject, String message){
   <span class="comments">// Send the email</span>
 }
}
</pre>

<pre>
package com.javacreed.examples.di.part1;

public class Person {
 private Email email = new Email();

 public void greetFriend(){
   email.sendEmail("Hello", "Hello my friend :)");
 }
}
</pre>

We all agree that this is a very simple and straightforward example that involves two simple Java classes.  Nevertheless, the above has some limitations as described below.

<ul>
<li>The <code>Person</code> class is dependent (has a strong/tight dependency) on the <code>Email</code> class.  There is a hard connection between these two classes.  Let say we have a new and better version of email class, <code>FastEmail</code>, in order for us to use it, we need to go in each and every class that depends on the <code>Email</code> class, such as the <code>Person</code> class, and replace it with the new version.</li>

<li>Let say we parameterise the <code>Email</code>'s constructor.  Again we have to go in each and every class that is initialising the <code>Email</code> class, such as the <code>Person</code> class, and change it.</li>

<li>A design decision is taken to make the <code>Email</code> class Singleton (<a href="http://en.wikipedia.org/wiki/Singleton_pattern" target="_blank">Wiki</a>).  Similar to above we need to modify all instances where it is used.</li>

<li>In order to improve the notifications/messages system, we decide to add different message delivery systems such as SMS or tweets.  The <code>Person</code> class and others like it, need to all be modified in order for it to use the new implementations.</li>

<li>Another developer needs to use the <code>Person</code> class, but would like to use a different notification/message system.  This cannot be achieved with the current version of the <code>Person</code> class as it is hardwired to the <code>Email</code> class.  What generally happens is that the other developer duplicates the <code>Person</code> class and modifies it as he/she needs.  The projects ends up with two versions of the <code>Person</code> class.</li>

<li>In the above points we mentioned many scenarios where code has to be changed.  All changes made, need to and should be tested.  How can we test the <code>Person</code> class without including the message delivery class such as the <code>Email</code>?  Testing, in many cases, is left as an afterthought.  The way we have the <code>Person</code> class constructed makes it hard to test it without involving the <code>Email</code> class.  Furthermore, how would we automate such test?  How can we use JUnit (<a href="http://junit.sourceforge.net/" target="_blank">Homepage</a>), or the like, to automate out tests?</li>

<li>Moving forward in the project, the <code>Person</code> class starts to depend on another class that allow this object to write a letter using the <code>Pen</code> class for example.  The <code>Person</code> class can use other ways to write a letter, such as <code>Pencil</code> class or <code>Typewriter</code> class, but this approach does not allow that.</li>
</ul>

These limitations can be improved by changing the way we think and refactor our code in a modular way.  This is independent from dependency injection and the dependency injection framework as we will see in the following section.

<h2>Change the way we think</h2>

The <code>Email</code> class provides a service, that is, sending of messages over the Internet using the mail protocol.  Instead of having the <code>Person</code> class initialising an instance of the <code>Email</code> class, we first create an interface, <code>MessageService</code>, and make the <code>Person</code> class using this interface instead.  This removes the dependency that the <code>Person</code> class has on the <code>Email</code> and replaces it with an abstract message delivery interface.

The following three steps: <em>define</em>, <em>implement</em> and <em>use</em>, show how we can develop modular and extendable code.  This approach also improves testing as we will see at the end of this article.

<ol>
<li><strong>Define Interfaces</strong> 

Many developers do not use interfaces as they see them as additional, non-required, code.  This may be true (I said maybe as the <code>System</code> class makes use of interfaces) for the famous <em>hello world</em> (<a href="/hello-world/" target="_blank">example</a>) program, definitely not true for the rest.  Like with everything else, we have to see things in context and there will be cases when we can do without interfaces.  Nevertheless, developing code using interfaces produce far more modular and extendable code as illustrated in the following examples.  The example discussed in the previous section was quite simple, but it included several pitfalls which can be easily avoided by using interfaces instead of concrete classes to define services.  Changing code at a later stage involves more work than having it right in the first place.

We start by defining the <code>MessageService</code> interface that includes one method, <code>sendMessage(String subject, String message)</code>.  For simplicity we assume that no exceptions are thrown.

<pre>
package com.javacreed.examples.di.part2;

public interface MessageService {
 void sendMessage(String subject, String message);
}
</pre>
</li>

<li><strong>Implement Interfaces</strong>

In the list of limitations we mentioned four possible methods of sending a message: email, fast email, SMS and tweet.  Let's create four classes that handle each message delivery method and have all these classes implement the interface created above. 
<pre>
package com.javacreed.examples.di.part2;

public class EmailService implements MessageService {
 @Override
 public void sendMessage(String subject, String message){
   System.out.printf("Email: %s, %s%n", subject, message);
 }
}
</pre>

<pre>
package com.javacreed.examples.di.part2;

public class FastEmailService implements MessageService {
 @Override
 public void sendMessage(String subject, String message){
   System.out.printf("Fast Email: %s, %s%n", subject, message);
 }
}
</pre>

<pre>
package com.javacreed.examples.di.part2;

public class SmsService implements MessageService {
 @Override
 public void sendMessage(String subject, String message){
   System.out.printf("SMS: %s, %s%n", subject, message);
 }
}
</pre>

<pre>
package com.javacreed.examples.di.part2;

public class TweetService implements MessageService {
 @Override
 public void sendMessage(String subject, String message){
   System.out.printf("Tweet: %s, %s%n", subject, message);
 }
}
</pre>
</li>

<li><strong>Use Interfaces</strong>

Finally, instead of using classes, we use interfaces.  In the <code>Person</code> class, we replace the <code>Email</code> field with the <code>MessageService</code> service interface as highlighted below.

<pre>
public class Person {
 <span class="highlight">private MessageService messageService;

 public Person(MessageService messageService){
   this.messageService = messageService;
 }</span>

 public void greetFriend(){
   messageService.sendMessage("Hello", "Hello my friend :)");
 }
}
</pre>

Note that the <code>Person</code> class is not initialising the message service but it is expecting it as a parameter of its constructor.  This is a key element in the design.  It improves modularity, extendibility and testing.  The <code>Person</code> class is not dependent on any implementation, but on a service defined by an interface.  This means that we can use the <code>Person</code> class without having to worry about the underlying implementation of the message service.  Furthermore, different <code>Person</code> instances can be instantiated using different message services.
</li>
</ol>

One can argue that the new version of <code>Person</code> class became more complex to instantiate as it requires parameters.  This is a fair statement and here is when dependency injection comes into play.

<h2>Using Dependency Injection</h2>

As mentioned in the introduction, dependency injection can help us initialising objects and provide these objects all the necessary resources (<em>ingredients</em>).  For example, the <code>Person</code> class requires an instance of <code>MessageService</code>.  The dependency injection framework will provide that for us.  So to create an instance of <code>Person</code> class, all we need to do is call something like: 

<pre>
dependecyFramework.getInstance(Person.class)
</pre>  

Magically, (not really), the dependency injection framework will create an instance of the <code>Person</code> class and provide an instance of the <code>MessageService</code> to the <code>Person</code> object.

The next natural question will be, how does the dependency injection framework knows how to initialise the <code>MessageService</code>?  We need to tell the dependency injection framework how to create an instance of <code>MessageService</code>.  With Guice we do that by creating a module (a class that extends <code>AbstractModule</code> class) as illustrated below.

<pre>
package com.javacreed.examples.di.part2;

import com.google.inject.AbstractModule;

public class ProjectModule extends AbstractModule {

 @Override
 protected void configure() {
   bind(MessageService.class).to(EmailService.class);
 }
}
</pre>

Here we are telling the dependency injection framework (Guice) how to create an instance of the <code>MessageService</code> class.  We also need to add an annotation to the <code>Person</code> class in order to allow the dependency injection framework to inject the necessary parameters.

<pre>
package com.javacreed.examples.di.part2;

import com.google.inject.Inject;

public class Person {
 private MessageService messageService;

 <span class="highlight">@Inject</span>
 public Person(MessageService messageService){
   this.messageService = messageService;
 }

 public void greetFriend(){
   messageService.sendMessage("Hello", "Hello my friend :)");
 }
}
</pre>

Note that the same concept can be applies to Spring or other dependency injection frameworks.  Each dependency injection framework has its way of configuration and the above only applies to Guice.  Spring and the others dependencies injection frameworks make use of different configuration methods.

With everything set, we create an instance of the <code>Person</code> class using the dependency injection framework.

<pre>
package com.javacreed.examples.di.part2;

import com.google.inject.Guice;
import com.google.inject.Injector;

public class Main {
 public static void main(String[] args) {
   Injector injector = Guice.createInjector(new ProjectModule());
   Person person = injector.getInstance(Person.class);
   person.greetFriend();
 }
}
</pre>

We replaced a couple of lines of code with many others.  What's the buzz about this?  In the next section we will see some of the benefits of dependency injection and how this can be used to simplify our coding life.

<h2>Benefits of Dependency Injection</h2>

In this section we will see some key benefits of dependency injection

<ul>
<li><strong>Changing the message service</strong>

Let's change the delivery method from email to SMS.  How would we do that?

We only need to change the <code>ProjectModule</code> class to map the <code>MessageService</code> class to the <code>SmsService</code> class as highlighted in the following example.

<pre>
package com.javacreed.examples.di.part3;

import com.google.inject.AbstractModule;

public class ProjectModule extends AbstractModule {

 @Override
 protected void configure() {
   bind(MessageService.class).to(<span class="highlight">SmsService.class</span>);
 }
}
</pre>

This one change will affect all classes initialised with the dependency injection framework without having to change any of these classes.  This leads us to another advantage: testing.
</li>

<li><strong>Testing and Mocking the message service</strong>

We can create a <code>MockMessageService</code> class which can be using in JUnit test case as shown next.

<pre>
package com.javacreed.examples.di.part3;

public class MockMessageService implements MessageService {

 public String subject;
 public String message;

 @Override
 public void sendMessage(String subject, String message) {
   this.subject = subject;
   this.message = message;
 }
}
</pre>

The above mock message service simply stores the parameters into two public fields.  These fields can be used to retrieve the values of the parameters and used for testing ass illustrated next.

<pre>
package com.javacreed.examples.di.part3;

import org.junit.Assert;
import org.junit.Before;
import org.junit.Test;

import com.google.inject.AbstractModule;
import com.google.inject.Guice;
import com.google.inject.Injector;
import com.google.inject.Singleton;

public class TestPerson {

 private Injector injector;

 @Before
 public void init() {
   injector = Guice.createInjector(new AbstractModule() {
     @Override
     protected void configure() {
       bind(MessageService.class).to(MockService.class);
     }
   });
 }

 @Test
 public void testGreetFriend() {
   Person person = injector.getInstance(Person.class);
   person.greetFriend();

   MockService mockService = injector.getInstance(MockService.class);
   Assert.assertEquals("Greet", mockService.subject);
   Assert.assertEquals("Hello my friend", mockService.message);
 }
}
</pre>

This may require some explanation.  So here we go.  In the <code>init()</code> method we created a dependency injection context just for this testing and provided a custom module (as an inner anonymous class).  The custom module wires the <code>MessageService</code> with the <code>MockMessageService</code> instead.  After the <code>greetFriend()</code> method is invoked, we ascertain that the correct parameters are being passed to the message service instance.

This design setup allows us to test the <code>Person</code> class independent from the other classes that it depends on in an automated manner.
</li>

<li><strong>Changing the signature of the <code>Person</code> constructor</strong>

As we mentioned in the list of limitations, the <code>Person</code> class may evolve and include more functionality.  Changing the signature of the <code>Person</code> constructor will not affect us as long as the injector knows how to provide the required parameters.

<pre>
package com.javacreed.examples.di.part3;

import com.google.inject.Inject;

public class Person {

 private final MessageService messageService;
 <span class="highlight">private final WritingService writingService;</span>

 @Inject
 private Person(final MessageService messageService, <span class="highlight">WritingService writeService</span>) {
   this.messageService = messageService;
   <span class="highlight">this.writingService = writeService;</span>
 }

 public void greetFriend() {
   messageService.sendMessage("Hello", "Hello my friend :)");
 }
 
 <span class="highlight">public void writeToFriend() {
   writingService.write("Hello my friend :)");
 }</span>
}
</pre>

The <code>Person</code> class will still be initialised in the same way as it is now.  Thus changing the <code>Person</code> constructor signature will not affect the other classes that make us of it.

<pre>
Person person = injector.getInstance(Person.class);
</pre>

The same applies for anything that is initialised and handled through the dependency injection framework.
</li>

<li><strong>Passing the <code>Injector</code> as parameter</strong>

In a project we can have one instance shared throughout the project.  This can be achieved by passing the <code>Injector</code> as a parameter to other objects that need it.  We setup the <code>Injector</code> at the beginning (in a <code>main()</code> method for example), and then have it set as a constructor parameter in all the classes that require an instance of the injector.  Like that, one injector will server all classes in the project.
</li>
</ul>

<h2>Conclusion</h2>

This ended up to be quite a long article.  Dependency injection is quite simple to use and it has quite a "<em>shallow</em>" learning curve.  This article did not explain how to use the actual dependency injection framework (such as Guice or Spring).  Articles about these are easily found.  This article described key benefits of using a dependency injection framework, even in small projects.

