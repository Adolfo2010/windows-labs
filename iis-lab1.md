## This Demo will deploy a Docker IIS Container on Windows 2016 Server.

### Requirements:

- Windows 2016 Server with at least 75GB on C:

- Docker For Windows

- Pull image microsoft/iis:nanoserver before executing this demo because you will need to download more than 1GB from Docker Hub.

__NOTE:__

[windows-nat-winnat-capabilities-and-limitations](https://blogs.technet.microsoft.com/virtualization/2016/05/25/windows-nat-winnat-capabilities-and-limitations)
~~~
There are some NAT limitations that we will have to managed during this lab.
SWe can not connect to localhost so we will need to review Docker Host IP Address and use your web browser using 
its IP as url.
~~~

### Docker Install on Windows 2016

1. Open an elevated PowerShell session


2. First install the OneGet PowerShell module.

~~~
	Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
~~~

3. Next you use OneGet to install the latest version of Docker.

~~~
	Install-Package -Name docker -ProviderName DockerMsftProvider
~~~

4. Reboot Server
	
~~~
	Restart-Computer -Force
~~~

5. Install Windows Updates

~~~
	sconfig --> Select 'Windows Update Settings' and 'Download all Updates'
~~~

6. Once all updates are installed, reboot again server

~~~
	Restart-Computer -Force
~~~

7. Deploy dotnet-sample "Hello World" Container for testing Docker installation

~~~
	docker run microsoft/dotnet-samples:dotnetapp-nanoserver
~~~

### Once Docker is installed we can start this Demo Lab

1. Create a container named "demoIIS" based on microsoft/iis image and expose container's port 80 on host's port 80.
	
~~~
	docker run -d --name demoIIS -p 80:80 microsoft/iis:nanoserver

	docker port demoIIS
~~~

2. We can now open our favorite browser on our PC and connect to host's IP (default por 80). We will get the "IIS Start" page.


3. Now we will  open a command line on running container and we will delete that starting page.

~~~
	docker exec -i demoIIS cmd

	del C:\inetpub\wwwroot\iisstart.htm
~~~

4. Now that we are connected and we have deleted the start page, let's create a new index page to show something when some ones gets our IIS Container on port 80.

~~~
	echo "TEST1 $(date)" > C:\inetpub\wwwroot\index.html
~~~


5. Now connect again to http://__HOST_IP__:80 and we will get the previously created index page.


6. Delete 'demoIIS' created container
	
~~~
	docker rm -fv demoIIS
~~~

7. Let's start again another container (we can use same name because 'demoIIS' doesn't exist). This time we will use port 1080 for publishing container's service.

~~~
	docker run -d --name demoIIS -p 1080:80 microsoft/iis:nanoserver
~~~

8. If we connect to http://__HOST_IP__:80 we will not get any service page because container's internal por 80 it isn't exposed on por 80. WE can connect to IIS on http://__HOST_IP__:1080

~~~
	docker port demoIIS
~~~

9. We will notice that we have again same "IIS Start" page, because new container is based on microsoft/iis image.


10. We will create a local index.html file and we will copy on running "demoIIS" container

~~~
	echo "TEST2 $(date)" > index.html 

	docker cp "$(pwd)\index.html" demoIIS:"c:\inetpub\wwwroot\index.html"
~~~

11. Now we will create a new image with our changes to index.html file (it didn't exist before on original base image). We need to stop running container to commit changes to new image.
 
~~~
	docker stop demoIIS

	docker commit demoIIS iis_changed:1.0
~~~

12. Let's start a new container based on previously create iis_changed:1.0 (don't forget tags!!!). Remember to publish container's service on a free port because 1080 can be used by running demoIIS (if you haven't deleted yet).

~~~
	docker run -d -p 2080:80  --name changed_demoIIS iis_changed:1.0
~~~

13. If we browse to http://__HOST_IP__:2080 we will get the modified index.html page. This is how we can easily make our changes permanent in a new image (or even on base image).


14. We now delete all containers and images created during this demo.

~~~
	docker rm -fv changed_demoIIS demoIIS

	docker rmi -f iis_changed:1.0
~~~
