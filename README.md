# krunnercommands

Command Line scripts for krunner utilizing the "run command" functionality krunner has. All commands have been scripted into their own file and corresponds with the file name. For example if you want to look into the code behind the `login` command then feel free to look at the source code in file `login`.

## How to execute these commands

On KDE use the shortcut ALT+Space to open krunner and then type in the command which has been provided by a file here.

## Install

-- To be continued --

## Command Documentation

### login

Opens the login url to a website, initiates a SSH connection or connects to a remote file system via sftp depending on the context you provide.

`login` and `remote` share the same codebase.

The `login` command works hand in hand with the [_standard unix password manager_](https://www.passwordstore.org/). It looks at your password store by utilizing `pass` which handles the store to find the name of the password entry you requested. Once found it will decrypt it and open the login url referenced there in firefox.

**Your story:** You have your GitHub credentials in your password store and the firefox addon "PassFF" installed (in firefox) and want to log in to Github to do good software coding stuff. And you're tired opening firefox and going to GitHub login page on your own. Glad that you installed the scripts here into your `PATH` and so you just do ALT+Space to open krunner and you type in `login github`. Magically but not surprisingly you will be prompted for your password store certificate password and if you enter it correctly then firefox will open with the GitHub login url in the address bar. If you now have `PassFF` firefox addon configured properly and only one login for this site then it should fill in the credentials on its own, hit the login button to log you finally in to do you a favor.

**Your second story:** You're an admin of thousand and one night servers and sick remembering the ssh commands to access each as you please or need. But you are a good admin and therefore stored each ssh command in an pgp file on its own. Glad that you installed the scripts here into your `PATH` and so you just do ALT+Space to open krunner and you type in `login personalserver` and say krunner that it should open you a terminal so you can interact with it. Magically but not surprisingly you will be prompted for your password store certificate password and if you enter it correctly then it opens the terminal displaying the ssh command it is going to execute. Now SSH does it usual job you love it for. The same applies also to sftp connections.

#### Syntax

`<name>` = Replace with the name of the gpg file in your password store. The search behind is **case-insensitive** and **recursive**. E.g. `login github` is the same as `login GitHub` as long as the pgp file has the name `GiThUb` or `gitHUB` etc. Never ever add the file extension `.pgp` because we're running Linux and can do better than our enemy users running Windows.

`login <name>`

Log in to a site: `login <name>` e.g. `login github` or `login GitHub`.

Log in to a server (only if executed in the foreground): `login <name>` e.g. `login personalServer` or `login Personalserver`.
Log in to a server and issue a single command: `login <name> <command to execute remotely (it is allowed that the command contains whitespaces but explicit quoting is not required)>` e.g. `login personalserver sudo apt update && sudo apt upgrade` or `login PersonalSERVER sudo apt update && sudo apt upgrade`.

Log in to a remote file system (only if executed in the background): `login <name>` e.g. `login personalServer` or `login Personalserver`.

You execute this script in the background if you call it in krunner without saying krunner to execute it in a terminal window. 

You execute this script in the foreground if you call it in krunner with saying krunner to execute it in a terminal window or if you call it directly from inside the terminal.

#### Internal logic

1. You use the syntax `login <name>` e.g. `login GitHub` or `login github` as long as you have a gpg file named `GitHub` or `github` or even `Github`.

2. The script lists all content in your password store using `tree` and performs a case-insensitive search for the `<name>` you provided.

3. When there is more than just one result it simply takes the first one.

4. It prepares the result for `pass` which requires a special notation for gpg password files.

5. `pass` asks you for the password of the certificate to be able to decrypt the content of the password file found and selected.

6. It determines the execution context:
   
   1. if executed in the background
      
      1. , a second argument provided to the `login` command and the `ssh` / `sftp` protocol being used in the `url` tag of the password file then it will initiate a **ssh connection** to the server to execute the command given as the second up until the last argument.
      
      2. the `ssh` / `sftp` protocol being used in the `url` tag of the password file then it will initiate a **sftp connection** to the server referenced in the url. Provided that the second argument will be left empty.
      
      3. and the `https` protocol being used in the `url` tag of the password file then it will open the url in firefox.
   
   2. if executed in the foreground
      
      1. a second argument provided to the `login` command and the `ssh` / `sftp` protocol being used in the `url` tag of the password file then it will initiate a **ssh connection** to the server to execute the command given as the second up until the last argument.
      
      2. the `ssh` / `sftp` protocol being used in the `url` tag of the password file then it will initiate a **ssh connection** to the server referenced in the url. Provided that the second argument will be left empty.
      
      3. and the `https` protocol being used in the `url` tag of the password file then it will open the url in firefox.

For it to work this script requires the password store to be at `~/.password-store` and decryption/enryption to be working. But also `pass` must display a graphical (GUI) prompt where you need to enter the password for your certificate you use for that password store.

If you want to just type `login github` your password file must have the name (file extension inclusive) `GitHub.pgp` or `github.pgp` or `gitHub.pgp` . But `Github-private.pgp` is then not allowed. So here the thumb rule comes:

- `login github` --> `GitHub.pgp` or `github.pgp` or `gitHub.pgp`

- `login <name>` --> `<Name>.pgp` or `<nAmE>.pgp` etc.

Please note the case-insensitive search!

##### GPG File containing url

Your pgp file e.g. for GitHub should have the following format:

```gpg
<your very very secure machine cryptographically generated password>
username: your_email@your_domain
url: https://github.com/login
```

##### GPG File containing ssh command

Your pgp file e.g. for your personal server should have at least the following format:

```gpg
<your very very secure machine cryptographically generated password>
url: ssh://your_username@your_domain
```

```gpg
<your very very secure machine cryptographically generated password>
url: ssh://your_username@your_domain:2705
```

or similiar.

### backupidle

The `backupidle` powers off or reboots your system if the following conditions are true

- Program `rsync`  not running (anymore)
  
  - `rsync` is a common tool used for backing up and keeping directories across different systems in sync. Since `rsync` syncs across systems, all files will be stored locally for offline use.

- Program `syncthing` not running (anymore) or if `syncthing` has finished syncronizing but is still running.
  
  - Just like `rsync` but provides a more modern look and allows easy syncronizarion across multiple systems with different operating systems without the need of an intermediate server. `syncthing` uses peer to peer technology to sync to the systems.

**Your story:** You came from vacation and already transfered all your photos to your laptop. Now you want to sync them to another workstation so you turn it on. You already set up syncthing on these two systems so they know and love each other. But you want the workstation to turn itself off when finished receiving the photos via syncthing so you execute `backupidle poweroff` on it.

#### Syntax

Power off system after syncronization has been finished: `backupidle poweroff`

Reboot system after syncronization has been finished: `backupidle reboot`

#### Internal logic

1. Retrieving API key from syncthing when installed and running

2. Periodically checking if `syncthing` is running and if yes then checking if it downloads files from another devices.

3. Periodically checking if `rsync` is running.

4. Running the action specified as argument when step 1 and 2 result both in `false`.

### updateidle

`updateidle` powers off or reboots your system if the following conditions are true

- Program `apt` not running

- Program `apt-get` not running

**Your story:** You currently update your system and this takes longer than you thought originally but with your eyes on the digital computer clock displayed in the middle of the topbar you realize that you need to leave the house now to get to your train. You want your computer to shut down when it finished the updates so you execute `updateidle poweroff` and run to your train.

#### Syntax

Power off system after updates has been installed: `backupidle poweroff`

Reboot system after updates has been installed: `backupidle reboot`

#### Internal logic

1. Periodically checking if `apt` is running.

2. Periodically checking if `apt-get` is running.

3. Running the action specified as argument when step 1 and 2 result both in `false`.



### firefoxidle

`firefoxidle` powers off or reboots your system if the following conditions are true

- Program `firefox` not running or

- `firefox` is not performing a file download currently

**Your story:** You download a big file from the world wide web using firefox, you look at the remaining hours and you take a clock on your clock just to realize that you need to get to university soon so you better prepare yourself going out. Mhhh, wait! What about the download? You definitely want your computer to be turned off when the download finishes. But who can do that? You live alone but luckily you have this great script. You already started the download, ensured that your internet connection remains stable for a successfull download and now you execute `firefoxidle poweroff` and leave your computer alone for its unattended power off.

#### Syntax

Power off system after download has been finished: `backupidle poweroff`

Reboot system after download has been finished: `backupidle reboot`

#### Internal logic

1. Scanning the home directory for files containing `.part` which are temporally files which firefox creates which grow in file size as firefox downloads data.

2. It only takes the first result so warching only one firefox download is possible

3. Periodically checking if `firefox` is running
   
   1. If not then execute desired command e.g. the one for powering off the system

4. and periodically checking if firefox finished downloading files.
   
   1. If finish then execute desired command e.g. the one for powering off the system.

### remote (deprecated)

`remote` and `login` share the same codebase. Superseded by `login`.

The `remote` command works hand in hand with the [_standard unix password manager_](https://www.passwordstore.org/). It looks at your password store by utilizing `pass` which handles the store to find the name of the password entry you requested. Once found it will decrypt it and ssh to a remote interactive terminal.

**Your story:** You're an admin of thousand and one night servers and sick remembering the ssh commands to access each as you please or need. But you are a good admin and therefore stored each ssh command in an pgp file on its own. Glad that you installed the scripts here into your `PATH` and so you just do ALT+Space to open krunner and you type in `remote personalserver` and say krunner that it should open you a terminal so you can interact with it. Magically but not surprisingly you will be prompted for your password store certificate password and if you enter it correctly then it opens the terminal displaying the ssh command it is going to execute. Now SSH does it usual job you love it for.

#### Syntax

`<name>` = Replace with the name of the gpg file in your password store. The search behind is **case-insensitive** and **recursive**. E.g. `remote personalserver` is the same as `remote PersonalServer` as long as the pgp file has the name `PersonalServer` or `pErSonALsErVer` etc. Never ever add the file extension `.pgp` because we're running Linux and can do better than our enemy users running Windows.

`remote <name> [<remote command>]`

Log in to a server: `remote <name>` e.g. `remote personalServer` or `remote Personalserver`.
Log in to a server and issue a single command: `remote <name> <command to execute remotely (it is allowed that the command contains whitespaces but explicit quoting is not required)>` e.g. `remote personalserver sudo apt update && sudo apt upgrade` or `remote PersonalSERVER sudo apt update && sudo apt upgrade`.

#### Internal logic

1. You use the syntax `remote <name> [<remote command>]` e.g. `login personalServer`, `login PersonalSERVER`, `remote personalserver sudo apt update && sudo apt upgrade` or `remote PersonalSERVER sudo apt update && sudo apt upgrade` as long as you have a gpg file named `PeRsOnAlserver` or `personalserver` or even `PersonalSERVER`.

2. The script lists all content in your password store using `tree` and performs a case-insensitive search for the `<name>` you provided.

3. When there is more than just one result it simply takes the first one.

4. It prepares the result for `pass` which requires a special notation for gpg password files.

5. `pass` asks you for the password of the certificate to be able to decrypt the content of the password file found and selected.

6. It filters the result so we have only the line with the ssh command.

7. It removes the string `ssh ` (note the whitespace at the end) and passes the rest (the ssh login command) over to SSH.

For it to work for e.g. being able to fetch the server login SSH command this script requires the password store to be at `~/.password-store` and decryption/enryption to be working. But also `pass` must display a graphical (GUI) prompt where you need to enter the password for your certificate you use for that password store.

If you want to type `remote personalserver` (with or without command parameter does not matter) your password file must have the name (file extension inclusive) `personalserver.pgp`, `PersonalServer.pgp` or `personalServer.pgp` . But `server-personal.pgp` is then not allowed. So here the thumb rule comes (with or without command parameter does not matter):

- `remote personalserver` --> `personalserver.pgp`, `PersonalServer.pgp` or `personalServer.pgp`

- `remote <name>` --> `<Name>.pgp` or `<nAmE>.pgp` etc.

Please note the case-insensitive search!

Your pgp file e.g. for your personal server should have the following format:

```gpg
<your very very secure machine cryptographically generated password>
username: your_username@your_domain
ssh your_username@your_domain
```

```gpg
<your very very secure machine cryptographically generated password>
ssh your_username@your_domain
```

```gpg
<your very very secure machine cryptographically generated password>
username: your_username@your_domain
ssh your_username@your_domain -P 2705
```

or similiar.

### flatinstall

**Your story:** You've got a fancy installation reference file from your best friend and stored it in your "Downloads" folder. Now you press ALT+Space and type in 'flatinstall flatpak' and let krunner open it in a konsole info so you can see the output. You approve the installation of that fancy tool. After installation the script moves the reference file from your Downloads to your trash folder. It is then up to you to decide to empty the trash and to delete that file for eternity from your hard drive. Now you have that fancy tool installed and you both can start doing things streight away.

#### Syntax

That commands requires you to say krunner it should execute it in the foreground. This command requires the flatpak reference file (`.flatpakref`) to be stored into your Downloads folder.

Install a flatpak app: `flatinstall`

#### Internal logic

1. You execute `flatinstall`
2. The script lists all content in your "Downloads" folder using `tree` and performs a case-insensitive search for a file with extension `.flatpakref`.
3. When there is more than just one result it simply takes the first one.
4. It prepares the result for `flatpak` which requires a special notation for their reference files.
5. It throws the ball of control over to flatpak
6. Flatpak throws the ball of control back and this script does the necessary cleanup.

For it to work it requires "~/Downloads" folder to exist and to be writeable. It also requires flatpak to be set up and the `.flatpakref` reference file you want to use for installation to be in the said folder because the script searches only there for it.   
