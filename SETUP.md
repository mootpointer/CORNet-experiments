# Getting Started

## Download and Setup VS Code

- Install Python Extension
- Install Jupyter Extension
- Install VS Code Remote SSH Extension

## Setup Cloud Resources

- Create an Azure for Students Account

### Download & Extract ImageNet

1. Open [Azure Cloud Shell](https://shell.azure.com)
2. Create a small (and cheap) VM with a big (and slow) disk.

   Run this command in the cloud shell to create the virtual machine:

   ```
   az vm create -g cornet -n downloader --image ubuntults \
   --generate-ssh-keys  --size Standard_B1ms \
   --data-disk-sizes 300 --storage_SKU Standard_LRS
   ```

3. SSH into your new VM to complete steps 4-?

   Find the public IP address for your new VM:

   ```
   az vm list-ip-addresses -g cornet -n downloader \
   --query "[].virtualMachine.network.publicIpAddresses[0].ipAddress" \
   --output tsv
   ```

4. Setup the data disk:

   ```
   sudo parted /dev/sda --script mklabel gpt mkpart xfspart xfs 0% 100%
   sudo mkfs.xfs /dev/sda1
   sudo partprobe /dev/sd
   sudo mkdir /imagenet
   sudo mount /dev/sda1 /imagenet
   sudo chown andrew /imagenet
   ```

5. Install `aria2` on the VM. This helps us download the dataset in parallel,
   which makes it faster. We're also installing `pv` while we're at it.
   ```
   sudo apt-get install aria2 pv
   ```
6. Download the training & validation sets with aria2c
   ```
   aria2c -x 16 <IMAGENET_TRAINING_URL>
   aria2c -x 16 <IMAGENET_VALIDATION_URL>
   ```
7. Extract the data:
   This will take a while, so I use `pv` so I can see progress. I'd do this
   inside tmux to make sure it sticks around even if the console closes.

   ```
   mkdir train && mv ILSVRC2012_img_train.tar train/ && cd train
   pv ILSVRC2012_img_train.tar | tar x
   rm ILSVRC2012_img_train.tar
   find . -name "*.tar" | while read NAME ; do mkdir -p "${NAME%.tar}"; tar -xvf "${NAME}" -C "${NAME%.tar}"; rm -f "${NAME}"; done
   cd ..
   mkdir val && mv ILSVRC2012_img_val.tar val/ && cd val && tar -xvf ILSVRC2012_img_val.tar
   wget -qO- https://raw.githubusercontent.com/soumith/imagenetloader.torch/master/valprep.sh | bash
   ```

- Launch GPU machine
- Move ImageNet disk to GPU machine
- Set Up Remote Dev
- Set up auto-shutdown
