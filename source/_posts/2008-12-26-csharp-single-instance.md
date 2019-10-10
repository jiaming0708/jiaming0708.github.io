---
title: '[C#] 單一化執行'
date: 2008-12-26 11:33:06
categories:
- Back-end
- C#
tags:
- C#
---
有的時候希望程式只單一執行，可以參考以下的程式

<!--more-->

如果你跟我一樣是一個需要登入的系統
用下面的方法會在開一次登入視窗
如果想要直接執行主視窗，可以用兩種方法
1. 起始的class為主視窗，這個方法我不太會
會知道這個方法是公司同事這樣寫

2. 起始的class為登入視窗，加上已經登入與否的判斷式在
``` csharp
    public partial class Login : Form
    { 
       [STAThread]
        static void Main()
        {
            try
            {
                //Application.Run(new Login());
                string[] args = Environment.GetCommandLineArgs();
                SingleInstanceController controller = new SingleInstanceController();
                controller.Run(args);
            }
            catch
            {
            }
        }

        public delegate void ProcessParametersDelegate(object sender, string[] args);

        public void ProcessParameters(object sender, string[] args)
        {
                this.Visible = true;
                this.TopMost = true;
                this.TopMost = false;
        }


        class SingleInstanceController : WindowsFormsApplicationBase
        {
            public SingleInstanceController()
            {
                // Make this a single-instance application
                this.IsSingleInstance = true;
                this.EnableVisualStyles = true;

                // There are some other things available in the VB application model, for
                // instance the shutdown style:
                this.ShutdownStyle = Microsoft.VisualBasic.ApplicationServices.ShutdownMode.AfterMainFormCloses;

                // Add StartupNextInstance handler
                this.StartupNextInstance += new StartupNextInstanceEventHandler(this.SIApp_StartupNextInstance);
            }

            /// <summary>
            /// We are responsible for creating the application's main form in this override.
            /// </summary>
            protected override void OnCreateMainForm()
            {
                // Instantiate your main application form
                // Create an instance of the main form and set it in the application; 
                // but don't try to run it.
                this.MainForm = new Login();

                // We want to pass along the command-line arguments to this first instance
                Random randNum = new Random();
            }

            /// <summary>
            /// This is called for additional instances. The application model will call this 
            /// function, and terminate the additional instance when this returns.
            /// </summary>
            /// <param name="eventArgs"></param>
            protected void SIApp_StartupNextInstance(object sender, StartupNextInstanceEventArgs eventArgs)
            {
                // Here you get the control when any other instance is 
                // invoked apart from the first one. 
                // You have args here in e.CommandLine.

                // You custom code which should be run on other instances

                // Copy the arguments to a string array
                string[] args = new string[eventArgs.CommandLine.Count];
                eventArgs.CommandLine.CopyTo(args, 0);

                // Create an argument array for the Invoke method
                object[] parameters = new object[2];
                parameters[0] = this.MainForm;
                parameters[1] = args;

                // Need to use invoke to b/c this is being called from another thread.
                this.MainForm.Invoke(new Login.ProcessParametersDelegate(((Login)this.MainForm).ProcessParameters), parameters);
            }
        }
    }
```