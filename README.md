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
## Library
