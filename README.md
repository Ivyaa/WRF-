# -WRF-
編譯紀錄/問題/解決方法/中文OuOb

這是一個編譯WRF的紀錄檔案，隨進度更新(目前測試通過Classic WRF及WPS編譯，還未到設定部分，正在嘗試Compression with NC4)
純粹紀錄，以下內容皆為本人在編譯過程中遇到的問題，包含環境變數(environment variable)及套件版本(module)等等  

※若有任何不正確的觀念歡迎提出🤣本人非資工專業，僅淺淺的學過一點東西  
※特別感謝身邊的大神學長、偉大的google/chatGPT  
※侵權刪(？)

# WRF版本
官網：https://www2.mmm.ucar.edu/wrf/users/download/get_source.html  
安裝教學：https://www2.mmm.ucar.edu/wrf/OnLineTutorial/compilation_tutorial.php#STEP1  
  
這兩個連結是官方提供的下載點以及所需library的編譯過程，WRF有許多不同版本，目前最高是WRF4.5，可以看自己需求去下載，但建議不要下載最新版本，可能會遇到很多編譯問題。這邊要提到的版本是Classic和Compression。由於一開始是完全按照官網的方式去執行，完全沒有注意到WRF library還有分Classic、Compression的差別🤣，這邊簡易的區分方式是否要使用NC4。若在編譯netcdf-C及netcdf-fortran時，加入選項--disable-netcdf-4，即是不使用NC4，理論上編譯出來的WRF就是Classic版本，這也是官網提供的版本。若不加入選項--disable-netcdf-4，編譯WRF的時候就會自動編譯成Compression版本，也會告訴你這個版本需要使用NC4。當然，這兩種版本還是會有其他區別，如果有興趣可以上官網或google看一下。  
  
會先提這個，是因為兩者對於library的要求不同，編譯library的順序、需要添加的選項也不一樣，在編譯的時候需要注意，不然就會像筆者一樣一個WRF編譯卡好久，但其實只是因為安裝netcdf-fortran的時候沒注意到這個問題。下面就以這兩種版本作區分來安裝WRF～

# Classic verion WRF4.4
先記錄一下使用環境和Linux版本  
linux : CentOS 7.9  
編譯器：icc/2023.1.0  

筆者最初是按照官網，在虛擬機自行裝CentOS7後，用GNU的C、FORTRAN編譯器(gcc、gfortran)，去做library的編譯，後來編譯換成intel了(icc、icpc、ifort)。Linux的部分，也有人選擇Ubuntu或是WSL，個人debug過程網路上看下來感覺Ubuntu是最多人選擇的，原因之一好像是CentOS系列對於編譯WRF來講不太友善(看過有人評論CentOS是對初編譯WRF來說最不友善的作業系統哈哈XD)。再來是不論使用何種編譯器、何種版本，一定要從頭到尾都用同一個版本的編譯器，不能說你用gcc7編譯library、用gcc10編譯WRF，請不要這樣做。
## Library
因為WRF會需要使用到netcdf-library(包含C和FORTRAN)以及其他相依的zlib、libpng、jasper，所以要先安裝並編譯library。另外若要支援平行化運算，就一定要安裝MPI這個工具。  
這邊想先提個，第一次編譯library，筆者也不太了解所謂「架構」，是被提了一下有不同「架構」的library，才開始了解官網在configure後面附加的一堆選項代表什麼意思。  
關於「架構」，筆者簡單理解為將編譯完的函式庫放入不同分類的資料夾，例如官網上的library架構可以大概圖解成：  

>WRFLIB_sharedstd  
>>grib2  
>>>bin  
>>>lib  
>>>include  
>             
>>netcdf  
>>>bin  
>>>lib  
>>>include 
> 

你會在configure後面看到：
>--prefix=path/to/grib2     #make install後會指定到此資料夾  

就是將編譯完成的函式庫放進這個資料夾中(會自行建立)，並進行簡單的分類(像是XXXX.h檔案會放在grib2/include/，這個資料夾底下；XXX.so檔案會放在grib2/lib/)  
筆者也有看到不同架構的函式庫，但是debug和設定環境變數可能會花很久就是了XD決定先照官網走。  
另外，從官網可以看到--disable-shared的選項，即是詢問你要不要設定成共享的函式庫，筆者最後是選擇設定「共享靜態函式庫」，所以在netcdf-c、netcdf-fortran編譯時，只會保留--disable-netcdf-4。  
再來是編譯library是有順序差異的，例如安裝netcdf-fortran前，一定要先安裝netcdf-c，不然會報錯。這邊可以按照官網的順序安裝，但筆者會按照以下順序去走(大神的智慧)：MPI >> zlib >> netcdf-c >> netcdf-fortran >> libpng >> jasper  

最後就是每一次編譯時，可以先看看每個library的安裝文件(INSTALL)，通常會寫安裝細節、可添加的選項以及可能會遇到的bug，事先閱讀減少debug時間。

### Environment variable
這個步驟屬實複雜，因為有太多環境參數可以調，主要是在編譯library和WRF/WPS時，告訴他們要使用到的工具在哪裡，要使用的工具(含版本)又是什麼。  
例如，筆者使用CentOS7，便含有內建的gcc4.8.5，但筆者想要使用intel icc/icpc/ifort，就要設定好如以下：  
>export CC="icc"  
>export CPP="icc -E"  
>export CXXCPP="icpc -E"  
>export FC="ifort -E"  
>export F77="ifort -E"
intel本身還有許多優化設定，

### Install zlib
由於筆者使用的環境有事先下載Intel mpi，已經包含mpi套件，沒有這個套件可以先下載來安裝。在安裝zlib(或是mpi/mpich)之前，強烈建議所有用GNU編譯器、並且是在CentOS7環境下的使用者，先想辦法升級gcc的版本，由於CentOS7原本的版本是gcc4.8.5(太老舊了)，有時候在後續安裝會缺少一些gcc後續版本會有的套件，如果後續才升級套件，又會碰到前面提到的用不同版本編譯器去編譯library的問題，直接砍掉重練，建議升到gcc7以上。  
安裝zlib過程基本上按照官網走，指定的資料夾是grib2。安裝完後，你會在你指定的路徑中找到grib2的資料夾，裡面包含bin/lib/include/share等等。接著，要透過設定環境變數LD_LIBRARY_PATH，將zlib函式庫路徑告知系統，以便後續安裝netcdf-c,fortran的時候，系統可以透過這個環境變數來找到需要的函式庫。
