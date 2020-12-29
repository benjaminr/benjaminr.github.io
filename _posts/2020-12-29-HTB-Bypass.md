---
title: "HTB: Bypass"
category: hacking
tags: htb sre hacking
---

> **Clue**: The Client is in full control. Bypass the authentication and read the key to get the Flag.

HTB: Bypass

After unzipping the archive, we have a single executable, _Bypass.exe_.
<!--ex-->
```bash
> unzip Bypass.zip
Archive:  Bypass.zip
[Bypass.zip] Bypass.exe password:
  inflating: Bypass.exe
```

I ran _file_ on the exe, just to double check it wasn't just conveniently named,

```bash
> file Bypass.exe
Bypass.exe: PE32 executable (console) Intel 80386 Mono/.Net assembly, for MS Windows
```

It looks like a genuine exe and in particular a .NET application. Googling around the topics of .NET decompilation and disassembly, I came across a few apps that looked helpful.

Most of the tools were Windows based, which is a problem for me, working on a Mac, so I thought it'd be a good opportunity to play around with AWS Workspaces.

## Setting up an AWS Workspace

For a general guide, see [here](https://docs.aws.amazon.com/workspaces/latest/adminguide/getting-started.html).

Navigate to the AWS Workspaces webpage and click _Get started with Amazon Workspaces_. 

Providing you are signed in you should be redirected to a regional dashboard where you can click _Get Started Now_.

![](/assets/images/HTB/Bypass/getting_started.png)

After that, select Quick Start and click the _Launch_ button.

![](/assets/images/HTB/Bypass/quick_start.png)

You'll then be prompted to select the Workspace you wish to launch. 

I selected _Standard with Windows 10_, as it was _Free tier_ eligible. 

The accompanying Directory should be automatically configured and shouldn't cost anything additional to the cost of the Workspace.

> If you use [Amazon WorkSpaces](https://aws.amazon.com/workspaces/), [Amazon WorkDocs](https://aws.amazon.com/workdocs/), or [Amazon WorkMail](https://aws.amazon.com/workmail/) in conjunction with AWS Directory Service, you will not be charged an additional fee for either Simple AD or AD Connector directories registered with these services, as long as you have active users of Amazon WorkSpaces, Amazon WorkDocs, or Amazon WorkMail. In order to qualify for free usage of Simple AD and AD Connector, you must have at least one active user for small directories each month and at least 100 active users for large directories each month.

![](/assets/images/HTB/Bypass/win10.png)

After entering a username, first name, last name and email address you should be able to launch the Workspace.

If you don't already have an account set up with a password, you should be emailed to set one up.

To log in to the Workspace for the first time, you will need to download a client for your operating system, or opt for the web client. These can be found [here](https://clients.amazonworkspaces.com/).

Grab the registration code from your email and ensure you've set up a password for your account by clicking through the link.

Once installed, launch the client and enter your credentials.

![](/assets/images/HTB/Bypass/workspace_login.png)

Providing everything goes smoothly, you should see a desktop appear:

![](/assets/images/HTB/Bypass/desktop.png)

## WorkDocs

If you want easy access to files from your own machine, I suggest also setting up WorkDocs. 

You'll first want to create one via the [AWS WorkDocs](https://aws.amazon.com/workdocs/) webpage. Opt for the _Quick Start_ and provide a unique name. You'll receive an accompanying email to set up your account once it has initialised. Make sure to do this before proceeding.

On your newly created AWS Workspace, click the Install Amazon WorkDocs icon and click through the installation until you're prompted by the following:

![](/assets/images/HTB/Bypass/workdocs.png)

After clicking _Get Started_, you'll be able to provide your unique WorkDocs URL and click next. You'll then be prompted for your username and password, which you should've just set up. 

Click through the menus and enable your preferred level of file syncing. 

You will now be able to upload files from your personal machine and have them appear in the Windows file system. Alternatively you could just login to your HTB account from your Workspace.

## Analysis

The first application I tried to analyse the exe was JetBrains dotPeek, but soon realised it can't be used to edit disassembled code. I ended up looking at dnSpy, which specifically focuses on debugging and .NET assembly editing.

After loading the exe into dnSpy and expanding the tree under HTBChallenge.exe, I opened up class _0_, which contained the following:

```csharp
using  System;  
  
// Token: 0x02000002 RID: 2  
public  class  0  
{  
	// Token: 0x06000002 RID: 2 RVA: 0x00002058 File Offset: 0x00000258  
	public  static  void  0()  
	{  
		bool  flag  =  global::0.1();  
		bool  flag2  =  flag;  
		if  (flag2)  
		{  
			global::0.2();  
		}  
		else  
		{  
			Console.WriteLine(5.0);  
			global::0.0();  
		}  
	}  
	  
	// Token: 0x06000003 RID: 3 RVA: 0x00002090 File Offset: 0x00000290  
	public  static  bool  1()  
	{  
		Console.Write(5.1);  
		string  text  =  Console.ReadLine();  
		Console.Write(5.2);  
		string  text2  =  Console.ReadLine();  
		return  false;  
	}  
	  
	// Token: 0x06000004 RID: 4 RVA: 0x000020C8 File Offset: 0x000002C8  
	public  static  void  2()  
	{  
		string  <<EMPTY_NAME>>  =  5.3;  
		Console.Write(5.4);  
		string  b  =  Console.ReadLine();  
		bool  flag  =  <<EMPTY_NAME>>  ==  b;  
		if  (flag)  
		{  
			Console.Write(5.5  +  global::0.2  +  5.6);  
		}  
		else  
		{  
			Console.WriteLine(5.7);  
			global::0.2();  
		}  
	}  
  
	// Token: 0x04000001 RID: 1  
	public  static  string  0;  
	  
	// Token: 0x04000002 RID: 2  
	public  static  string  1;  
	  
	// Token: 0x04000003 RID: 3  
	public  static  string  2  =  5.8;  
}
```

By placing breakpoints in method 0.0 on line:

```csharp
		if  (flag2)  
```

And method 0.2 on line:

```csharp
		if  (flag)  
```

I was able to stop the execution and change the flag values to true, forcing the execution of the following line:

```csharp
			Console.Write(5.5  +  global::0.2  +  5.6);  
```
I found the application was exiting before I could retrieve the flag, so I added an additional breakpoint at the end of 0.2 to halt the flow.

```bash
	Enter a username: ben
	Enter a password: ben
	Please Enter the secret Key: test
	Nice here is the Flag:HTB{}
```
As you can see, the values entered in the application can be anything you want, so long as the conditions are met.
