# Mac Development Ansible Playbook

This playbook installs and configures most of the software I use on my Mac for web and software development. Some things in macOS are slightly difficult to automate, so I still have a few manual installation steps, but at least it's all documented here.

## Installation

1. Ensure Apple's command line tools are installed (`xcode-select --install` to launch the installer).
2. [Install Ansible](https://docs.ansible.com/ansible/latest/installation_guide/index.html):

   1. Run the following command to add Python 3 to your $PATH: `export PATH="$HOME/Library/Python/3.8/bin:/opt/homebrew/bin:$PATH"`
   2. Upgrade Pip: `sudo pip3 install --upgrade pip`
   3. Install Ansible: `pip3 install ansible`

3. Clone or download this repository to your local drive.
4. Run `ansible-galaxy install -r requirements.yml` inside this directory to install required Ansible roles.
5. Run `ansible-playbook main.yml --ask-become-pass` inside this directory. Enter your macOS account password when prompted for the 'BECOME' password.

> Note: If some Homebrew commands fail, you might need to agree to Xcode's license or fix some other Brew issue. Run `brew doctor` to see if this is the case.

### Use with a remote Mac

You can use this playbook to manage other Macs as well; the playbook doesn't even need to be run from a Mac at all! If you want to manage a remote Mac, either another Mac on your network, or a hosted Mac like the ones from [MacStadium](https://www.macstadium.com), you just need to make sure you can connect to it with SSH:

1. (On the Mac you want to connect to:) Go to System Preferences > Sharing.
2. Enable 'Remote Login'.

> You can also enable remote login on the command line:
>
>     sudo systemsetup -setremotelogin on

Then edit the `inventory` file in this repository and change the line that starts with `127.0.0.1` to:

    ```
    [ip address or hostname of mac]  ansible_user=[mac ssh username]
    ```

If you need to supply an SSH password (if you don't use SSH keys), make sure to pass the `--ask-pass` parameter to the `ansible-playbook` command.

### Running a specific set of tagged tasks

You can filter which part of the provisioning process to run by specifying a set of tags using `ansible-playbook`'s `--tags` flag. The tags available are `dotfiles`, `homebrew`, `mas`, `extra-packages` and `osx`.

    ansible-playbook main.yml -K --tags "dotfiles,homebrew"

## Overriding Defaults

Not everyone's development environment and preferred software configuration is the same.

You can override any of the defaults configured in `default.config.yml` by creating a `config.yml` file and setting the overrides in that file. For example, you can customize the installed packages and apps with something like:

    ```yaml
    homebrew_installed_packages:
      - git
      - go

    mas_installed_apps:
      - { id: 926036361, name: 'LastPass Password Manager' }
      - { id: 425424353, name: 'The Unarchiver' }
      - { id: 441258766, name: 'Magnet' }
      - { id: 775737590, name: 'iA Writer' }
      - { id: 1116599239, name: 'NordVPN IKE - Unlimited VPN' }
      - { id: 1278508951, name: 'Trello' }
      - { id: 462058435, name: 'Microsoft Excel' }
      - { id: 462062816, name: 'Microsoft PowerPoint' }
      - { id: 497799835, name: 'Xcode' }

    gem_packages:
      - name: bundler
        state: latest

    npm_packages:
      - name: webpack

    pip_packages:
      - name: mkdocs

    configure_dock: true
    dockitems_remove:
      - Launchpad
      - TV
    dockitems_persist:
      - name: 'Visual Studio Code'
        path: '/Applications/Visual Studio Code.app/'
        pos: 5
    ```

Any variable can be overridden in `config.yml`; see the supporting roles' documentation for a complete list of available variables.

## Future additions

### Things that still need to be done manually

It's my hope that I can get the rest of these things wrapped up into Ansible playbooks soon, but for now, these steps need to be completed manually (assuming you already have Xcode and Ansible installed, and have run this playbook).

1. Set JJG-Term as the default Terminal theme (it's installed, but not set as default automatically).
2. Install all the apps that aren't yet in this setup (see below).
3. Remap Caps Lock to Escape (requires macOS Sierra 10.12.1+).
4. Set trackpad tracking rate.
5. Set mouse tracking rate.
6. Configure extra Mail and/or Calendar accounts (e.g. Google, Exchange, etc.).

### Configuration to be added

I have vim configuration in the repo, but I still need to add the actual installation:

    ```sh
    mkdir -p ~/.vim/autoload
    mkdir -p ~/.vim/bundle
    cd ~/.vim/autoload
    curl https://raw.githubusercontent.com/tpope/vim-pathogen/master/autoload/pathogen.vim > pathogen.vim
    cd ~/.vim/bundle
    git clone git://github.com/scrooloose/nerdtree.git
    ```

## Testing the Playbook

Many people have asked me if I often wipe my entire workstation and start from scratch just to test changes to the playbook. Nope! Instead, I posted instructions for how I build a [Mac OS X VirtualBox VM](https://github.com/geerlingguy/mac-osx-virtualbox-vm), on which I can continually run and re-run this playbook to test changes and make sure things work correctly.

Additionally, this project is [continuously tested on GitHub Actions' macOS infrastructure](https://github.com/geerlingguy/mac-dev-playbook/actions?query=workflow%3ACI).
