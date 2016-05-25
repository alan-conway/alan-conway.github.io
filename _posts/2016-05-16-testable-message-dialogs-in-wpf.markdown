---
layout: post
title:  "Testable Message Dialogs in WPF"
date:   2016-05-16 11:45:23
categories: posts
tags: ['#TDD', '#xUnit', '#Moq', '#WPF']
---
`#TDD #xUnit #Moq #WPF`

This posts continues the discussion of my reversi project. Source code can [be found here](https://github.com/alan-conway/Reversi) and if you'd like to download and play the game then you can do so by [clicking here](https://ci.appveyor.com/api/projects/alan-conway/reversi/artifacts/Reversi.zip?branch=master&job=Configuration%3A+Release)

I wanted to add a popup message in my game to allow the player to confirm they want to start a new game and hadn't clicked the button by mistake.

Simply adding a call to `MessageBox.Show` from within the ViewModel would not be a good idea because not only would it be difficult to test, it would also potentially stop other tests from completing.

The solution I used was to create a simple `IMessageDialogService` interface which looks a bit like this:

~~~ C#
public interface IMessageDialogService
{
	YesNoEnum ShowYesNoMessageDialog(string caption, string message);
}
~~~

Now instead of having the ViewModel make a call to `MessageBox.Show`, it can call instead to the instance of `IMessageDialogService` which was injected as a dependency.  For testing, we can mock the service and return whichever value we choose, and when we run the application for real, it's a simple matter to create a concrete instance of the interface that shows a popup dialog.
