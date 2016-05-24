---
layout: post
title:  "AutoFixture and auto mocking"
date:   2016-05-25 11:22:14
categories: posts
tags: ['#TDD', '#xUnit', '#Moq', '#AutoFixture']
---
`#TDD #xUnit #Moq #AutoFixture`

[Source Code](https://github.com/alan-conway/Reversi)

[Download and play](https://ci.appveyor.com/api/projects/alan-conway/reversi/artifacts/Reversi.zip?branch=master&job=Configuration%3A+Release)

In my [first post](posts/reversi-with-tdd.html) discussing my reversi project, I pointed out that I'm using TDD with xUnit and Moq, and highlighted a fluent builder pattern that I was using to build some of the objects that I would be testing.  
Since writing that post, I have been reading and learning more about AutoFixture and its automocking capabilities, and have now ported my test code to use this instead.

### _AutoFixture:_  
At its heart, AutoFixture tries to help simplify the 'Arrange' stage of unit testing.  
One way in which it does so is by providing a convenient way to create anonymous data to feed into tests.  
To understand what that means, let's look at a trivial example.  
Suppose our system under test is an Address Book and we have a test such as the following for adding a new address into the address book:

~~~ C#
[Fact]
public void ShouldAddNewAddressIntoAddressBook()
{
	//Arrange
	string newAddress = "29 Acacia Road";
	var addressBook = new AddressBook();

	//Act
	addressBook.Add(newAddress);

	//Assert
	Assert.Equal(1, addressBook.Addresses.Count);
}
~~~

In this case, the value of the new address that we are adding is not important to us - it could be anything and the test would remain the same.  
So in situations such as these we can use AutoFixture to create some dummy data for us, rather than having to put it together ourselves.

~~~ C#
[Fact]
public void ShouldAddNewAddressIntoAddressBook()
{
	//Arrange
	var fixture = new Fixture();
	string newAddress = fixture.Create<string>();
	var addressBook = new AddressBook();

	//Act
	addressBook.Add(newAddress);

	//Assert
	Assert.Equal(1, addressBook.Addresses.Count());
}
~~~

In this new example, we get a random string returned to us (eg `05a41b95-34b8-4b4b-920a-b3fc92e62dc0`) which this is used to give our address some value.  
AutoFixture can also do this with other data types, including our own custom types, and this is where it becomes very useful. Suppose that instead of adding a `string` into our `AddressBook` we instead add a `Person` into our `ContactsList`, where `Person` is a class that we have defined in our application. Then the code for our test will look essentially the same but AutoFixture will be doing much more for us:

~~~ C#
[Fact]
public void ShouldAddNewPersonIntoContactList()
{
	//Arrange
	var fixture = new Fixture();
	string newPerson = fixture.Create<Person>();
	var contacts = new ContactList();

	//Act
	contacts.Add(newPerson);

	//Assert
	Assert.Equal(1, contacts.People.Count());
}
~~~

The code here is just as convenient to write as the case for a simple string, but here our Person object might have a complex definition, such as:

~~~ C#
public class Person
{
	public string FirstName { get; set; }
	public string LastName { get; set; }
	public DateTime DateOfBirth { get; set; }
	public Address HomeAddress { get; set; }
	public List<EmailAddress> EmailAddresses { get; set; }
}
~~~

Even with non-trivial members, AutoFixture would supply us with a Person object with values populated in every field. The `HomeAddress` field would be an object of type `Address` (assuming that that is defined elsewhere in our project) with values for all of its fields, and the `EmailAddresses` field would be a populated collection with multiple items in it.

### _AutoFixture with xUnit:_
xUnit provides a way of running the same test more than once with different sets of input through its `[Theory]` attribute.  
AutoFixture, with its available extra xUnit nuget package, allows this to be extended to automatically supply anonymous data objects into tests.  

~~~ C#
[Theory]
[InlineAutoData(1)]
[InlineAutoData(2)]
public void ShouldAddEmployeeToDirectory(int employeeId, Employee employee)
{
		....
}
~~~

In this example, the usual `[InlineData]` attribute has become `[InlineAutoData]` and we are still able to supply a value for all the fields if we chooose to. But if we do not then AutoFixture will automatically supply anonymised data for the remaining arguments.

### _AutoFixture with AutoMoq:_

With an additional extra nuget package, AutoFixture can also become an automocking container which will perform dependency injection and will automatically create mocks of interfaces by default.  For example, suppose we have code such as this  

~~~ C#
public interface IFoo
{
	int Foo();
}

public interface IBar
{
	double Bar();
}

public class Worker : IWorker
{
	public Worker(IFoo foo, IBar bar)
	{
		...
	}
}
~~~

Then as long as we tell AutoFixture that we want to automatically mock the interfaces, then we can do the following:

~~~ C#
[Fact]
public void ShouldDoWork()
{
	var fixture = new Fixture().Customize(new AutoMoqCustomization());
	var worker = fixture.Create<Worker>();
	...
}
~~~

This will automatically create an instance of `Worker`, and will have done so by injecting mocked instances of `Foo` and `Bar` as constructor arguments.

If we want to Setup one or both of the interfaces then we can do so as follows:

~~~ C#
[Fact]
public void ShouldDoWork()
{
	var fixture = new Fixture().Customize(new AutoMoqCustomization());
	var mockFoo = fixture.Freeze<Mock<IFoo>>();
	mockFoo.Setup(f => f.Foo())
	       .Returns(123);
	var worker = fixture.Create<Worker>();
	...
}
~~~

The `Freeze` method here tells AutoFixture that once it has created its mock of `IFoo` then it should return that same instance each time it resolves that interface, rather than returning a different instance each time.


### _Further Reading:_

Find out more about AutoFixture by checking out the [Cheat Sheet](https://github.com/AutoFixture/AutoFixture/wiki/Cheat-Sheet) or [this list of blog posts](http://blog.ploeh.dk/tags/#AutoFixture-ref) from Mark Seemann, the author of AutoFixture
