# Using Globus 



pip install globus_cli
# -> opens in browser (login with your globus account)
globus login

globus endpoint search --filter-scope my-endpoints

# install globus personal https://docs.globus.org/how-to/globus-connect-personal-linux/
wget https: // downloads.globus.org / globus - connect - personal / linux / stable / globusconnectpersonal - latest.tgz
tar xzf globusconnectpersonal-latest.tgz
cd globusconnectpersonal-3.2.2
./globusconnectpersonal -setup -no-gui
# -> save the local endpoint uuid to a variable
monolyth=5e0cbed2-61e4-12ee-e80f-7b7665fb6de1

start transfer
globus transfer -v ba993270-4516-48b9-af7c-77e890865733:/license.txt $monolyth:/home/twak/winsyn
# save the task id 

# view progress of transfers here
https://app.globus.org/activity
or on the command line:
globus task show 24f95d56-61e6-11ee-b9d7-11fee6a821c0
