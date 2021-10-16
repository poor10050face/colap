
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
