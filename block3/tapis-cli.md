
Tutorial: Getting Started with Tapis CLI
===============================================
The following instructions will guide you through setting up Tapis CLI.  As an aside, everything we do today can also be accomplished from a command line interface or by directly calling API endpoints.  

The Tapis CLI commands all respond with help for -h and return back information on the parameters that can be passed.  

Need help?  Ask your questions using the [TACC Cloud Slack Channel](https://bit.ly/2XHYJEk)

Initial Requirements
===============================================

Before getting started, you need to have the following:
* A TACC Account - today you have a test account
* SSH access to the Stampede 2 compute cluster and an allocation.
* Familiarity with [editing text files](https://www.nano-editor.org/dist/v2.7/nano.html) and [working at the command line](http://www.gnu.org/software/bash/manual/bashref.html#Introduction)

Any questions?  Join the [TACC CLOUD SLACK CHANNEL](https://bit.ly/2XHYJEk) and ask away.


Command Line Access
===================

We won't install it in this workshop (since it is already installed on the VM), but everything we do today can also be done from the standard shell using the Tapis CLI tools.  

Open a terminal in your Jupyter instance and that is what we will use to run the CLI commands.

Jump to the [Authentication](https://github.com/tapis-project/uh-hpc-in-the-cloud/blob/master/block3/tapis-cli.md#authentication) section for the workshop.

Installing the Tapis CLI Tools (Skip for Workshop- this is at home )
------------------------------

Tapis has a downloadable set of command line tools that make it easier to work with the API from the shell. Using these scripts is generally easier than hand-crafting cURL commands, but if you prefer that route, consult the [Tapis API Documentation](https://tacc-cloud.readthedocs.io/en/latest/). We include these scripts in the training virtual machines and supplement them with additional support scripts, example files, and documents.

During the course, we will use the Jetstream Cloud virtual machines, but if you have a shell on your personal computer, you can install these tools on your own later.

To use access the CLI for this tutorial you can open a Terminal  in Jupyter which give you access to the shell in the Jetstream VM, OR *ssh* into the system from you own terminal:

```ssh ubunut@jetstreamVM_ip_address```

Install the CLI tools (Skip for Workshop- this is at home )
----------------------------------

The CLI tools and instructions for installation can be found in the [CLI repository](https://github.com/TACC-Cloud/agave-cli)


Authentication
----------------

Tapis has robust Authentication/Authorization pathways - we could easliy spend an hour or more discussing them, but will keep our focus simple for this tutorial.

The Tapis API uses OAuth 2 for managing authentication and authorization. OAuth 2 is an open standard for access delegation, commonly used as a way for Internet users to grant websites or applications access to their information on other websites but without giving them the passwords.

Just understand that instead of passing a username and password every time we want to make an authenticated/authorized request to the Tapis APIs we will be usig an Access Token that has a defined expiration - this keeps our credentials safe and ensures that if someone where to obtain the token it could not be used forever.

Run the following in the CLI
```
>auth-check
Please run /agave-cli/bin/tenants-init to initialize your client before attempting to interact with the APIs.
```
We will see that we have to initialize some things before we can use Tapis.

Initialize the CLI
------------------

The first time you install the CLI tools on a computer, you need to initialize it.
You can initialize the TACC tenant by runnning:

```
> auth-session-init
ID                   NAME                                     URL
vdjserver.org        VDJ Server                               https://vdj-agave-api.tacc.utexas.edu/
sgci                 Science Gateways Community Institute     https://sgci.tacc.cloud/
iplantc.org          CyVerse Science APIs                     https://agave.iplantc.org/
sd2e                 SD2E Tenant                              https://api.sd2e.org/
3dem                 3dem Tenant                              https://api.3dem.org/
designsafe           DesignSafe                               https://agave.designsafe-ci.org/
araport.org          Araport                                  https://api.araport.org/
tacc.prod            TACC                                     https://api.tacc.utexas.edu/
irec                 iReceptor                                https://irec.tenants.prod.tacc.cloud/
agave.prod           Agave Public Tenant                      https://public.agaveapi.co/
bridge               Bridge                                   https://api.bridge.tacc.cloud/
portals              Portals Tenant                           https://portals-api.tacc.utexas.edu/

Please specify the ID for the tenant you wish to interact with: tacc.prod
Creating a client...
API username: train100
API password:
Created client 5c8c91edb474 - Autogenerated client
Getting oauth bearer tokens...
```
Select the 'tacc.prod' tenant and then use the username and password provided for this tutorial.

The 'auth-session-init' command creates a Tapis client and then request an API token and will then place the TACC tenant,client and API token information into a cache in ~/.agave/current. This is the file that the CLI tools will look for when making API calls so that you don't have to enter those parameters for every call.


Creating a Client 
----------------
The Tapis API uses OAuth 2 for managing authentication and authorization. Before you work with Tapis, you must create an OAuth client application and record the API keys that are returned. This is a one-time action per machine that you use the CLI on and the 'auth-session-init' can take care of this.  In the event you need to create your own client you can pass additional parameters to the 'auth-session-init' command.  For instance if we want to make a new client.

```
> auth-session-init -h
usage: auth-session-init [-h] [-c CACHEDIR] [--tenants TENANTS] [-t TENANT]
                         [-u USERNAME] [-N CLIENT_NAME] [-D DESCRIPTION]

Create a new Agave oauth client

optional arguments:
  -h, --help            show this help message and exit
  -c CACHEDIR, --cachedir CACHEDIR
                        Directory to save confiurations in.
  --tenants TENANTS     URL with tenants listings.
  -t TENANT, --tenant TENANT
                        Tenant id for session.
  -u USERNAME, --username USERNAME
                        Session username.
  -N CLIENT_NAME, --name CLIENT_NAME
                        Name of client.
  -D DESCRIPTION, --description DESCRIPTION
                        Description of client.

> auth-session-init -N myclient1
Client 'myclient1' is not saved in /root/.agave, so we will create it...
Creating a client...
API password:
Created client myclient1 - Autogenerated client
Getting oauth bearer tokens...
API password:
```

*Note:* The -N flag allows you to specify a machine-readable name for your client and -D provides the description. 

You will need access to the ```consumerKey``` and ```consumerSecret``` values when setting up on other hosts. So, please take a moment and record *client_name*, *consumerKey*, and *consumerSecret* somewhere safe. If you lose these values, you can create a new instance of the client by deleting the old client (clients-delete CLIENT_NAME) and creating it again (or create a new client with a different name).

 OAuth 2 API authentication token
----------------

Tokens are a form of short-lived, temporary authenticiation and authorization used in place of your username and password. To interact with Tapis, you will need to acquire one. Each Tapis token, typically, expires after 4 hours, but can easily be refreshed.

On a host where you have configured a Tapis OAuth2 client already, the CLI command to get a new token is:

```
> auth-tokens-create -v
API password:
```

You will then be prompted to enter your *API password*. **Type your user password**.  At this point, you should receive an affirmation of success in your terminal that resembles this one:

```
Token for tacc.prod:train100 successfully refreshed and cached for 13605 seconds
{
  "scope": "default",
  "token_type": "bearer",
  "expires_in": 13605,
  "refresh_token": "fd38287337b5312933eea555555",
  "access_token": "f940624e12e7186117443ee555555"
}
```

NOTE that the CLI will cache the new access and refresh tokens in the ~/.agave/current file.

## Refreshing your token

This tutorial won't take very long, but if you are interrupted and come back later, you might find your token has expired. You can always refresh a token as follows:

```> auth-tokens-refresh -v```

A successful refresh should appear:

```
Token for tacc.prod:train100 successfully refreshed and cached for 14400 seconds
{
  "scope": "default",
  "token_type": "bearer",
  "expires_in": 14400,
  "refresh_token": "b4b5c3e5b7c77862af8088c4a92c6a25",
  "access_token": "561335afda3b35654c84dc6d483f7ccf"
}
```

This topic is covered in great detail at the Tapis [Authorization Guide](https://tacc-cloud.readthedocs.io/projects/agave/en/latest/agave/guides/authorization/introduction.html)

NOTE that most CLI commands will attempt to do a token refresh on your behalf if the access token is expired.

## Command Help

Note that all the CLI commands take the '-h' flag to display a short description and the accept parameters for the command.


```python

```
