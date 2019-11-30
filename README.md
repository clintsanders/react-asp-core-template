# react-asp-core-template
Template project for new React web applications with Web API backend

The project was created by executing the following command in the root project directory
  `dontnet new react`
  
## Configure IIS Express to use self-signed certificate
Usually on the dev machine you will get the error 

>`This site can't be reached localhost refused to connect` 

This is because the certifcate needs to be create in IIS Express for the port you will be debugging the projecct on.  To configure this for a new project first close Visual Studio and delete the .vs folder. This folder and config files will be re-created when you launch Visual Studio again.

Open cmd with admin privileges in the directory C:\Program Files (x86)\IIS Express and execute the following command
`iisexpressadmincmd.exe setupsslurl -url:https://localhost:12345/ -useselfsigned` 

The port number for HTTPS used by visual studio when launching IIS Express can be found in the launchSettings.json under the Properties folder.

Relaunch Visual Studio and you should be able view the index.html page in your browser.
