$aptsetup = <<-SCRIPT
echo Setting up apt
sudo sed -i 's|http://in.archive.ubuntu.com/ubuntu|http://cz.archive.ubuntu.com/ubuntu|g' /etc/apt/sources.list
sudo apt-get clean
cd /var/lib/apt
sudo mv lists lists.old
sudo mkdir -p lists/partial
sudo apt-get clean
sudo apt update -y && sudo apt upgrade -y && sudo apt autoremove -y
echo -e "\n\n=== RELOADING ==="
SCRIPT

$installtools = <<-SCRIPT
echo -e "\n\n=== BASIC APT INSTALL ==="
sudo apt install -y apt-transport-https wget build-essential libncursesw5-dev libssl-dev gpg libsqlite3-dev tk-dev libgdbm-dev libc6-dev libbz2-dev libffi-dev zlib1g-dev software-properties-common ca-certificates curl gnupg lsb-release jq gnupg software-properties-common build-essential checkinstall unzip python3 python3-pip
echo -e "\n\n=== GitHub ==="
curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | sudo dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg \
&& sudo chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg \
&& echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null \
&& sudo apt update \
&& sudo apt install gh -y
echo -e "\n\n=== AWS ==="
AWSTMP=$(mktemp -d) && cd $AWSTMP && \
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip" && \
unzip awscliv2.zip && \
sudo ./aws/install
echo -e "\n\n=== Terraform ==="
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update
sudo apt install -y terraform
terraform --version
SCRIPT

Vagrant.configure("2") do |config|
    config.vm.box = "bento/ubuntu-22.04"
    config.vm.network "forwarded_port", guest: 8080, host: 8080
    config.vm.provider "virtualbox" do |v|
        v.memory = 4096
        v.cpus = 2
    end
    config.vm.provision "aptsetup", before: :all, type: "shell", inline: $aptsetup, reboot: true
    config.vm.provision "installTools", type: "shell", inline: $installtools
    config.vm.provision "gitconfig", type: "file", source: ".gitconfig", destination: ".gitconfig"
    config.vm.provision "personalize", type: "shell", after: :all, path: "secrets.sh"
end
