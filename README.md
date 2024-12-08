# WarpAttack: Bypassing CFI through Compiler-Introduced Double-Fetches
WarpAttack is a new attack vector that exploits compiler-introduced double-fetch
optimizations to mount TOCTTOU attacks and bypass code-reuse mitigations. 

See our [paper](http://hexhive.epfl.ch/publications/files/23Oakland3.pdf) for more details.

Citing WarpAttack:
```
@inproceedings{xu2023warpattack,
  title={WarpAttack: Bypassing CFI through Compiler-Introduced Double-Fetches},
  author={Jianhao Xu and Luca Di Bartolomeo and Flavio Toffalini and Bing Mao and Mathias Payer},
  booktitle={2023 IEEE Symposium on Security and Privacy},
  year={2023},
  organization={IEEE}
}
```
## Gadget code detection
We provide a lightweight [binary analysis tool](./gadget_detection/gadget.py) based on Radare2 to detect WarpAttack gadgets. 

Prerequisites
- Python 3.6 or later. You can download from the [official website](https://www.python.org/downloads/). 
- Radare2. You can download the latest version of Radare2 from the [official website](https://rada.re/n/)
- Python packages. You can get all the packages through `pip install r2pipe click bisect`.

Usage
- To use this script, you need to provide a list of input files to be analyzed via stdin. The input files should be separated by space or newlines. You can use the following command to run the script:
```
cat input_files.txt | python3 gadget.py output_file.txt
```
Another example to analyze all possible executable files under one folder:
```
find path/to/target_folder -type f ! -size 0 -exec grep -IL . "{}" \; | python3 gadget.py output_file.txt
```

## Proof of Concept exploit
To get arbitrary Read&Write, we introduce an out-of-bound bug to Firefox 106.0.1 
inspired from one [CTF challenge](https://devcraft.io/2018/04/27/blazefox-blaze-ctf-2018.html). 
Please find the patch [here](./poc_exploit/blaze.patch). 
Note that if you would like to reproduce the exploit with the same gadget we use, please use GCC to compile the Firefox.
Please note that if you intend to reproduce the exploit using the same gadget that we used, you should compile Firefox with GCC.

We also provide a [web page](./poc_exploit/warpattack1a.html) containing malicious [JS code](./poc_exploit/warpattack1a.js). The exploit will be 
triggered when the web page is accessed with the vulnerbale Firefox browser.


-----
You can run this experiment on a fedora 36 VM.
1. download virtual box, install fedora 36 iso, you can download it from [direct download link](https://archives.fedoraproject.org/pub/archive/fedora/linux/releases/36/Workstation/x86_64/iso/Fedora-Workstation-Live-x86_64-36-1.5.iso) After you finish the installing, remember to remove the fedora-workstation-live-36-1.5.iso from the virtual box. Watch youtube video [here](https://www.youtube.com/watch?v=AZzLpOicH-8). 

2. install fedora 36 on virtual box, with ideally 8gb ram, 100gb storage and 8 cpu cores.
3. inside fedora36 virtual box, download this google drive folder [RPMS of firefox installer](https://drive.google.com/file/d/1rMucTRtS6g2iH0k4Utn5pepJ33YAgFeH/view?usp=drive_link). Extract the RPMS.tar.xz by running `tar -xvf RPMS.tar.xz` in terminal.
4. remove the old firefox by running `sudo dnf remove firefox` in terminal. 
5. in terminal run `sudo dnf install /path/to/firefox-106.0.1-1.fc36.x86_64.rpm` to install blaze firefox 106.0.1 which is the vulnerable version with patch.
6. ***DISABLE ASLR*** by running `echo 0 | sudo tee /proc/sys/kernel/randomize_va_space` in terminal.
7. clone the git and goes to folder `cd WarpAttack/poc_exploit/`
8. run ./test4core.sh to run the expirment!!!

## IMPORTANT
We don't have a success double fetch attack yet. No stack smashing detected.
