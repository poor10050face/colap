
import os

username = "user" #@param {type:"string"}
password = "root" #@param {type:"string"}

print("Creating User and Setting it up")

# Creation of user
os.system(f"useradd -m user")

# Add user to sudo group
os.system(f"adduser user sudo")
    
# Set password of user to 'root'
os.system(f"echo 'user:root' | sudo chpasswd")

# Change default shell from sh to bash
os.system("sed -i 's/\/bin\/sh/\/bin\/bash/g' /etc/passwd")

print("User Created and Configured")







#@title **RDP**
#@markdown  It takes 4-5 minutes for installation

import os
import subprocess

#@markdown  Visit http://remotedesktop.google.com/headless and Copy the command after authentication

CRP = "DISPLAY= /opt/google/chrome-remote-desktop/start-host --code=\"4/0AX4XfWhCY3CdbDQPEhc5u3qKfeVjmZXapW8CvCOf1tZJL3qWf5QRvIhNdSZ-4yOSFRW7XA\" --redirect-url=\"https://remotedesktop.google.com/_/oauthredirect\" --name=$(hostname)" #@param {type:"string"}

#@markdown Enter a pin more or equal to 6 digits
Pin = 123456 #@param {type: "integer"}


class CRD:
    def __init__(self):
        os.system("apt update")
        self.installCRD()
        self.installDesktopEnvironment()
        self.installGoogleChorme()
        self.finish()

    @staticmethod
    def installCRD():
        print("Installing Chrome Remote Desktop")
        subprocess.run(['wget', 'https://dl.google.com/linux/direct/chrome-remote-desktop_current_amd64.deb'], stdout=subprocess.PIPE)
        subprocess.run(['dpkg', '--install', 'chrome-remote-desktop_current_amd64.deb'], stdout=subprocess.PIPE)
        subprocess.run(['apt', 'install', '--assume-yes', '--fix-broken'], stdout=subprocess.PIPE)

    @staticmethod
    def installDesktopEnvironment():
        print("Installing Desktop Environment")
        os.system("export DEBIAN_FRONTEND=noninteractive")
        os.system("apt install --assume-yes xfce4 desktop-base xfce4-terminal")
        os.system("bash -c 'echo \"exec /etc/X11/Xsession /usr/bin/xfce4-session\" > /etc/chrome-remote-desktop-session'")
        os.system("apt remove --assume-yes gnome-terminal")
        os.system("apt install --assume-yes xscreensaver")
        os.system("systemctl disable lightdm.service")

    @staticmethod
    def installGoogleChorme():
        print("Installing Google Chrome")
        subprocess.run(["wget", "https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb"], stdout=subprocess.PIPE)
        subprocess.run(["dpkg", "--install", "google-chrome-stable_current_amd64.deb"], stdout=subprocess.PIPE)
        subprocess.run(['apt', 'install', '--assume-yes', '--fix-broken'], stdout=subprocess.PIPE)

    @staticmethod
    def finish():
        print("Finalizing")
        os.system(f"adduser {username} chrome-remote-desktop")
        command = f"{CRP} --pin={Pin}"
        os.system(f"su - {username} -c '{command}'")
        os.system("service chrome-remote-desktop start")
        print("Finished Succesfully")


try:
    if username:
        if CRP == "":
            print("Please enter authcode from the given link")
        elif len(str(Pin)) < 6:
            print("Enter a pin more or equal to 6 digits")
        else:
            CRD()
except NameError as e:
    print("username variable not found")
    print("Create a User First")




#@title **Google Drive Mount**
#@markdown Google Drive used as Persistance HDD for files.<br>
#@markdown Mounted at `user` Home directory inside drive folder
#@markdown (If `username` variable not defined then use root as default).


def MountGDrive():
    from os import environ as env
    from google.colab import drive

    config = env['CLOUDSDK_CONFIG']
    addr = env['TBE_CREDS_ADDR']

    ! runuser -l $user -c "yes | python3 -m pip install --user google-colab"  > /dev/null 2>&1

    mount = """from os import environ as env
from google.colab import drive

env['CLOUDSDK_CONFIG']  = '{config}'
env['TBE_CREDS_ADDR'] = '{addr}'

drive.mount('{mountpoint}')""".format(config=config, addr=addr, mountpoint=mountpoint)

    with open('/content/mount.py', 'w') as script:
        script.write(mount)

    ! runuser -l $user -c "python3 /content/mount.py"

try:
    mountpoint = f"/home/{username}/drive"
    user = username
except NameError:
    print("username variable not found, mounting at `/content/drive' using `root'")
    mountpoint = '/content/drive'
    user = 'root'

MountGDrive()



#@title **Google Drive Mount**
#@markdown Google Drive used as Persistance HDD for files.<br>
#@markdown Mounted at `user` Home directory inside drive folder
#@markdown (If `username` variable not defined then use root as default).


def MountGDrive():
    from os import environ as env
    from google.colab import drive

    config = env['CLOUDSDK_CONFIG']
    addr = env['TBE_CREDS_ADDR']

    ! runuser -l $user -c "yes | python3 -m pip install --user google-colab"  > /dev/null 2>&1

    mount = """from os import environ as env
from google.colab import drive

env['CLOUDSDK_CONFIG']  = '{config}'
env['TBE_CREDS_ADDR'] = '{addr}'

drive.mount('{mountpoint}')""".format(config=config, addr=addr, mountpoint=mountpoint)

    with open('/content/mount.py', 'w') as script:
        script.write(mount)

    ! runuser -l $user -c "python3 /content/mount.py"

try:
    mountpoint = f"/home/{username}/drive"
    user = username
except NameError:
    print("username variable not found, mounting at `/content/drive' using `root'")
    mountpoint = '/content/drive'
    user = 'root'

MountGDrive()





#@title **SSH**

! pip install colab_ssh --upgrade &> /dev/null

Ngrok = True #@param {type:'boolean'}
Agro = False #@param {type:'boolean'}


#@markdown Copy authtoken from https://dashboard.ngrok.com/auth (only for ngrok)
ngrokToken = "1x6gMQwCHdhjGTMe34KZ9j24jqC_7ERBerYzrNbKhQ7LELe6X" #@param {type:'string'}


def runNGROK():
    from colab_ssh import launch_ssh
    from IPython.display import clear_output
    launch_ssh(ngrokToken, password)
    clear_output()

    print("ssh", username, end='@')
    ! curl -s http://localhost:4040/api/tunnels | python3 -c \
            "import sys, json; print(json.load(sys.stdin)['tunnels'][0]['public_url'][6:].replace(':', ' -p '))"


def runAgro():
    from colab_ssh import launch_ssh_cloudflared
    launch_ssh_cloudflared(password=password)


try:
    if username:
        pass
    elif password:
        pass
except NameError:
    print("No user found using username and password as 'root'")
    username='root'
    password='root'


if Agro and Ngrok:
    print("You can't do that")
    print("Select only one of them")
elif Agro:
    runAgro()
elif Ngrok:
    if ngrokToken == "":
        print("No ngrokToken Found, Please enter it")
    else:
        runNGROK()
else:
    print("Select one of them")
    
    
    
    #@title **Colab Shutdown**

#@markdown To Kill NGROK Tunnel
NGROK = True #@param {type:'boolean'}

#@markdown To Unmount GDrive
GDrive = False #@param {type:'boolean'}

#@markdown To Sleep Colab
Sleep = False #@param {type:'boolean'}

if NGROK:
    ! killall ngrok

if GDrive:
    with open('/content/unmount.py', 'w') as unmount:
        unmount.write("""from google.colab import drive
drive.flush_and_unmount()""")
    
    try:
        if user:
            ! runuser $user -c 'python3 /content/unmount.py'
    except NameError:
        print("Google Drive not Mounted")

if Sleep:
    from time import sleep
    sleep(43200)
