# Using Globus 

These are my notes on getting a local download running with Globus. First we must install the libraries, then create a local endpoint, then intialize a transfer from the WinSyn endpoint to your local endpoint.

```
pip install globus_cli
globus login
```
this opens in browser (login with a globus account)

Then install globus personal, as  [https://docs.globus.org/how-to/globus-connect-personal-linux/](https://docs.globus.org/how-to/globus-connect-personal-linux/) and create a local endpoints.
```
wget https://downloads.globus.org/ globus - connect - personal / linux / stable / globusconnectpersonal - latest.tgz
tar xzf globusconnectpersonal-latest.tgz
cd globusconnectpersonal-3.2.2
./globusconnectpersonal -setup -no-gui
```

Save the output of the above command - it's the uuid for your local endpoint. It will be different for you!
```
local_endpoint=5e0cbed2-61e5-15ef-e80f-abcdeffb6de1
```

While we're at it, let's put the WinSyn endpoint into a variable. This is the same for everyone
```
winsyn=ba993270-4516-48b9-af7c-77e890865733
```

Finally, we can start the transfer of the dataset (all 4Tb of it) to `/home/twak/winsyn` on your local machine:
```
globus transfer $winsyn:/ $local_endpoint:/home/twak/winsyn
```

You can view the progress [online](https://app.globus.org/activity). Keep a note of the task id to view the status on the command line:
```
globus task show 23425d56-ffff-11ee-b9d7-11fee6a821c0
```

or even cancel it:
```
globus task cancel 23425d56-ffff-11ee-b9d7-11fee6a821c0
```
