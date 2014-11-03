---
layout: post
title: "The missing Lines method in .NET"
date: 2014-11-03 20:39:12 +0200
comments: true
categories: 
---

In my .NET project I needed a method to split a multi-line string into multiple lines. Ruby has such a method out of the box. Here is an example:

``` 
my_multi_line_text = %q{my
	 »  multiline
	 »  text}
my_multi_line_text.lines
```

here is an output:

``` 
[
[0] "my\n",
[1] "multiline\n",
[2] "text"
]
```

Lets go ahead and implement “Lines” extension method in .NET. Lets attack it via **TDD approach**.

At first we need to create two Class Library projects in Visual Studio. First – Extensions and Second Extensions.Tests:

 "Visual Studio window with two projects"

Lets add Nunit package to our test project:

"Launching Nuget"

Here it is:

"Nunit via Nuget"

For my tests I’m going to use a notation from fabulous Roy Osherove’s book [“The Art of Unit Testing in .NET”](http://www.manning.com/osherove/). The gist of the notation is that the following naming convention is used for the test methods:

**[Method Under Test]_[Context]_[Expected_Result]**

it’s time to write our first test:
```
[TestFixture]
class StringExtensionsTests
{
[Test]
public void Lines_EmptyString_ReturnsEmptyList()
{
	//Arrange & Act
	var lines = "".Lines();

	//Assert
	Assert.IsEmpty(lines);
}
}
```

Now lets write just enough code to get both projects compiled:

```
public static List<string> Lines(this string input)
	{
		return null;
	}

```

The test will fail for sure. That’s ok. I want to see my tests failed first (I don’t want to have an illusion that my code works. I want to make it working). Ok lets go ahead and make it green:

```
 public static List<string> Lines(this string input)
		{
			if (string.IsNullOrEmpty(input))
			{
				return new List<string>();
			}
			return null;
		}
```

Now let check against Null String:

```
 [Test]
	public void Lines_NullString_ReturnsEmptyList()
	{
		//Arrange & Act
		string nullInput = null;
		var lines = nullInput.Lines();

		//Assert
		Assert.IsEmpty(lines);
	}

```

Ok we are good to go and implement a test for a single line string:

```
[Test]
public void Lines_OneLineString_ReturnsListWithOneElement()
{
	//Arrange & Act

	var lines = "Test Message".Lines();

	//Assert
	Assert.AreEqual(lines.Count, 1);
	Assert.AreEqual(lines.First(), "Test Message");
}
```

Lets make this test green by adding some extra logic:

```
public static List<string> Lines(this string input)
	{
		if (string.IsNullOrEmpty(input))
		{
			return new List<string>();
		}
		else
		{
			return input.Split(new String[] { Environment.NewLine },StringSplitOptions.None).Select(s=>s.Trim()).ToList();
		}
	}

```

Now we can write couple of tests to be sure that multi-line strings work as well:

```
[Test]
public void Lines_MultiLineString_ReturnsListWithMultipleElements()
{
	//Arrange & Act
	string text = @"Test Message1
					Test Message2
					Test Message3";

	var lines = text.Lines();

	//Assert
	Assert.AreEqual(lines.Count, 3);
	Assert.AreEqual(lines[0], "Test Message1");
	Assert.AreEqual(lines[1], "Test Message2");
	Assert.AreEqual(lines[2], "Test Message3");
}


[Test]
public void Lines_YetAnotherMultiLineString_ReturnsListWithMultipleElements()
{
	//Arrange & Act
	string text = @"Hello,
					World!!!";

	var lines = text.Lines();

	//Assert
	Assert.AreEqual(lines.Count, 2);
	Assert.AreEqual(lines[0], "Hello,");
	Assert.AreEqual(lines[1], "World!!!");
}
```

The last two tests violate **DRY** principle, since they do kind of the same with different parameters though.

Nunit framework can easily resolve that issue via **TestCase** attribute, so we can provide multiple test cases for the same testing code:

```
  [TestCase("Test Message1\r\nTest Message2\r\nTest Message3", new string[] {"Test Message1","Test Message2","Test Message3"})]
	[TestCase("Hello,\r\nWorld!!!", new string[] { "Hello,", "World!!!" })]
	public void Lines_MultiLineString_ReturnsListWithMultipleElements(string text, string [] expectedLines)
	{
		var lines = text.Lines();
		Assert.AreEqual(lines, expectedLines.ToList());
	}

```

Knowing that we can refactor null/empty string tests and combine them into one test:

```
[Test]
	[TestCase(null)]
	[TestCase("")]
	public void Lines_NullOrEmptyString_ReturnsEmptyList(string inputString)
	{
		var lines = inputString.Lines();
		//Assert
		Assert.IsEmpty(lines);
	}
```

That's it. In my opinion Roy Osherove's **[Method Under Test] __[Context] __[Expected_Result]** naming notation is great. At least I haven't seen anything better for .NET world yet.
After writing specs for piles of Ruby code using Rspec I'm really in love with its syntax though. So In the next post I'm going to leverage Rspec and IronRuby to prove a concept that we can cover .NET code with tests written in Ruby.

