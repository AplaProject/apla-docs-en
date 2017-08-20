################################################################################
Installation and configuration
################################################################################

This section discusses the installation and configuration of EGAAS.

********************************************************************************
Installation on Windows
********************************************************************************

Let's consider how the test version of EGAAS is installed on Windows.

1.	Go to https://golang.org/dl/ and download the last stable version of golang compiler depending on the bit version your computer is running. For example, if the last version is 1.8.3, then from the **Stable Versions** list, download the file go1.8.3.windows-386.msi if you have a 32-bit Windows, and go1.8.3.windows-amd64.msi for a 64-bit Windows.

2.	Launch this installation on your computer. The default installation directory is c:\Go and it is better not to change it. To check that everything has been successfully installed, run the *go version* command, which will show the current version.

3.	Create a directory on any of your disks, where golang projects will be stored. For example, C:\MyGo. After that, launch the **setx GOPATH c:\MyGo** command, which will point your directory as a folder with your go projects. Launch the go env command, which will display a list of parameters. Check to make sure that your directory is listed in *GOPATH*. Actually, you can skip this step, but then you must use the directory to which *GOPATH* points by default when running *go env*.

4.	Go to https://www.enterprisedb.com/downloads/postgres-postgresql-downloads, download the PostgreSQL database. Select the latest version there and specify Windows x86-32 or Windows x86-64 as your Windos OS, depending on the bit version your computer is running.

5.	Launch the downloaded PostgreSQL installation file. You can leave the default installation path as it is. But specify the path for storing the database as *c:\PostgreSQL*. You can specify *postgres* as a password for a super user; you can leave the default values of all other parameters as they are. In the end, you will be asked to launch Stack Builder in order to install additional components – you can skip this step.

6.	Go to https://sourceforge.net/projects/mingw-w64/ and download mingw-w64-install.exe. Launch it. If you have a 32-bit Windows, leave the installation options unchanged, but if you have a 64-bit version, change Architecture from *i686* to x86_64. Go to the directory where mingw is installed and find the file mingw-w64.bat. In the second line, the path to bin is indicated, such as *C:\Program Files\mingw-w64\x86_64-6.1.0-posix-seh-rt_v5-rev1\mingw64\bin*. Copy this path to the clipboard. In Windows, go to Control Panel → System → About System → Advanced system settings. Then select the Additional tab and click Environment Variables below. In the section System Variables, find the PATH variable and select it. Click Edit. Add the path you copied here. Also add the bin directory where PostgreSQL is installed, for example, *c:\Program Files\PostgreSQL\9.6\bin*.

7.	Go to https://git-scm.com/download/win and download the relevant (32 or 64) version of Git for Windows and launch this installation. All other options can be left unchanged during installation.

8.	Now launch the **go get -u github.com/EGaaS/go-egaas-mvp** command. Execution of the command can last for several minutes. After execution, you should have the directory c:\MyGo\src\github.com\EGaaS\go-egaas-mvp with source files. There is another way to get the source files – go to github.com/EGaaS/go-egaas-mvp. Click Clone & Download and click Download ZIP. After that, just unpack the contents of the go-egaas-mvp-master directory of this zip into the directory *c:\MyGo\src\github.com\EGaaS\go-egaas-mvp*.

9.	Launch the **go get -u github.com/jteeuwen/go-bindata** command and then go to the directory *c:\MyGo\src\github.com\jteeuwen\go-bindata\go-bindata*. Launch **go install**. The file go-bindata.exe will be created in the *c:\MyGo\bin* directory

10.	Go to the directory *c:\MyGo\src\github.com\EGaaS\go-egaas-mvp* and launch the file *bindata.sh* or you can launch **C:\MyGo\bin\go-bindata -o="packages/static/static.go" -pkg="static" static/....** After that, execute the **go install** command in the same place. If everything is set up properly, the file *go-egaas-mvp.exe* will be created in the directory c:myGobin.

11.	Copy go-egaas-mvp.exe to a directory, for example, *c:\EGAAS* and launch it. Your browser will open the page http://localhost:7079 with installation options. To test it locally on your blockchain, select Private-net. To specify a DB Name (Database Name), you need to first create it. To do this, launch **psql -U postgres** and enter your postgres password below. After the *postgres=#* prompt, type **CREATE DATABASE egaas** and click Enter. After that, enter *egaas* in the installation settings, and enter postgres as DB user and DB password. Then click on the install button. If successful, you will be prompted to restart EGAAS. Find the program icon in the system tray and select Exit with right mouse click. After that, launch the program again. Installation of EGAAS is completed.
