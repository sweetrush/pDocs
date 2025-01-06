## Setup the Python Virutal Enviroment with Anaconda 
- Download the anaconda Install stript from [Download Script](https://www.anaconda.com/download)
- You can running the script in the Directory by Running the following Commends 
```bash
> sh Name_of_Anaconda_InstallationScript_.sh 
```
This will install the Anaconda Package to your Linux System. 
- Once install you can now runn commands lile 

#### Checking the Current Version
```bash
    conda -V # This is to check for the Version that you are running 
```

#### Creating a Virtual Enviroment 

- The Following command is the way to create a virtual enviroment in Anaconda 
```sh
# conda create -n Folder_Name/EnviromentName python=PYTHON_VERSION anaconda
# Following Example Below. 
# Note the Version of the Python or your Virutal Enviroment 

conda create -n AppsTest python=3 anaconda 

```
- Next you will need to activate your Virutal enviroment to access it and start developing in it. Use the following command while following example noted above. 
```sh
  
  > conda activate AppsTest 
  (AppsTest) Development_Director/>

```

- To deactivate this Virual Enviroment do the following 
```sh 
 (AppsTest) Development_Director> conda deactivate 
  
```
