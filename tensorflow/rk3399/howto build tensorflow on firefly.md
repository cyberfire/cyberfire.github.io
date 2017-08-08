Tensorflow uses bazel building system to create the binaries, which will download the necessary packages from internet automatically.
However, when building tensorflow on firelfy (RK3399) board, there are several issues to be resolved.

### 1. Usethe newest Bazel  for ARM architecture detection
### 2. Insert an external USB stick as memory swap partition
### 3. Patch Eigen to fix compilation of Jacobi rotations with ARM NEON
### 4. Set resource limits on bazel to build

Here is the detailed steps to build tensorflow from source on firefly

## 1. Install Bazel from source

   Please visit https://docs.bazel.build/versions/master/install-compile-source.html for official guide.
   
   Install openjdk: 
   
       sudo apt-get install openjdk-8-jdk
   
   Download bazel release: https://github.com/bazelbuild/bazel/releases/download/0.5.2/bazel-0.5.2-dist.zip
   
   Unzip the archive and call bash ./compile.sh. Copy the generated output/bazel to /usr/bin
   
## 2. Add the memory swap partition

   Insert one USB stick (>=2G) on the USB port and suppose it is /dev/sdx
   
   Format it as swap partition:  
   
      mkswap /dev/sdx
      
      This command should output the UID XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX. If that is not the case, please use blkid to get the UID.
    
   Add the swap entry in /etc/fstab: add one line in /etc/fstab
   
      UUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX none swap sw,pri=5 0 0
   
   Enable swap by: 
   
      swapon -a

## 3. Configure tensorflow
   ./configure 
   Just select the default options.
      
## 4. Patch Eigen 
   The patch is included here and is from: https://bitbucket.org/eigen/eigen/commits/d781c1de9834/
   
   Download all external packages needed first: bazel fetch //tensorflow/examples/label_image
   
   Change directory to where Eigen is unpacked:
    cd ~/.cache/bazel/_bazel_xxx/xxxxxxx/external/eigen_archive
    
    patch -p1 < eigen.patch
   

## 5. Use below command to build tensorflow
	 bazel build -c opt  --local_resources 1024,1.0,1.0 --verbose_failures //tensorflow/examples/label_image

## 6. Try label_image
    
   If every thing goes well, after about one hour, label_image should be created at bazel-bin/tensorflow/examples/label_image.
   
   Download the model data and the sample photo by:
   
     curl -L "https://storage.googleapis.com/downoad.tensorflow.org/models/inception_v3_2016_08_28_frozen.pb.tar.gz" | tar -C tensorflow/examples/label_image/data -xz
     
   Now, please run:
   
     bazel-bin/tensorflow/examples/label_image/label_image   



   
   
   
   
   

   
   
   