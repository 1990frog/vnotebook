Importing SSL certificates into svnX
I just wanted to connect to my subversion repository with svnX via the https protocol. Guess what, svnX did not want to connect because it did not trust my selfsigned certificate. The only way to continue was to dismiss the error message with no way to import the certificate.

Turns out that svnX is using the command line svn client to communicate with repositories. So you need to mark the SSL certificate as trusted in the command line client and then svnX will work automatically. To do that, fire up a terminal window and just list the contents of your repository:

```
svn list https://whatever.server.com/repository/
```

Now svn will print a certificate validation error and ask whether you want to dismiss or accept the cerificate. Accept it permanently to get rid of that warning. Now you can access the repository via svnX.

