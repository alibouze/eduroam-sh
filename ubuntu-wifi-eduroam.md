# Ansluta en Ubuntu server till eduroam med WiFi

För att ansluta till eduroam via WiFi från din ubuntu server krävs lite konfiguration. Följande guide gäller uppkoppling mot eduroam på BTH (Blekinge tekniska högskola).  
Eduroam är ett s.k enterprise nätverk (PEAP).  

+ Se till att ditt WiFi-kort eller WiFi-dongel fungerar med drivrutin etc.
Ta reda på namnet på ditt nätverkskort (dongel). Vanligen: `wlan0`
Ladda ner certifikatet för att få ansluta till BTHs nätverk och spara det på lämplig plats.

`$ sudo wget -O /etc/ssl/certs/AddTrust-External-CA-Root.cer https://raw.githubusercontent.com/mattische/eduroam-sh/master/AddTrust-External-CA-Root.cer`

+ Redigera filen `/etc/wpa_supplicant/wpa_supplicant.conf` med en texteditor (exemplen nedan använder vim och nano).  
      Om den inte finns - skapa den.  
      `$ sudo nano /etc/wpa_supplicant/wpa_supplicant.conf` 



Skriv följande i filen:

      network={  
      ssid="eduroam"  
      key_mgmt=WPA-EAP  
      auth_alg=OPEN  
      eap=PEAP  
      ca_cert="/etc/ssl/certs/AddTrust-External-CA-Root.cer"  
      identity="ditt användarnamn"  
      phase2="auth=MSCHAPV2"  
      password="ditt lösenord"  
      }  

+ Redigera filen `/etc/networks/interfaces` och skriv in följande i filen (om det inte redan står). Innan du redigerar bör du ta reda på vad ditt nätverkskort heter. Nedan används `wlan0`, men det kan heta något annat t ex `wlp1s0`. För att ta reda på vad ditt heter kan du kolla output från `ifconfig` eller `iwconfig`. Om det heter något annat, byt ut `wlan0` till aktuellt namn.  
**Om du har ett fast/statiskt ip** (vanligtvis via ethernet):  

      `auto wlan0`  
      `allow-hotplug wlan0`  
      `iface wlan0 inet manual`  
      `wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf`  

  OBS!
  **Om du INTE HAR ETT STATISKT IP**, dvs ovanstående gäller eller fungerar inte - byt ut till att få ett ip via dhcp;

      auto wlan0  
      allow-hotplug wlan0  
      iface wlan0 inet dhcp  
      wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf  
      
  Kör sedan;

      $ sudo ifdown wlan0 && ifup wlan0`
      testa sedan t ex genom att pinga;
      `$ ping -c3 www.google.com

+ Till sist - testa att reboota för att kolla så att allt funkar.   
Med kommandot ifconfig kan du kolla om ditt nätverkskort har fått en IP-adress av DHCP-servern. Leta efter "wlan0" (eller vad ditt nätverkskort heter) i outputen.

`$ sudo ifconfig`

Du kan behöva;  
`$ sudo ifup wlan0` 


# utan eduroam  
Dvs ett 'vanligt' nätverk (icke enterprise).  
Om du blir tilldelad ett ip via dhcp behöver du endast editera `/etc/network/interfaces` genom att lägga till något i stil med;  

            auto wlan0  
            iface wlan0 inet dhcp
            wpa-ssid namn_på_nätverket
            wpa-psk lösenord_för_nätverket  

och sedan förmodligen göra en `$ sudo ifdown wlan0 && ifup wlan0`

<br>
Om du har ett ip du ska använda editerar du samma fil något i stil med;  

      auto wlan0
      iface wlan0 inet static
      address 192.168.1.150
      netmask 255.255.255.0
      gateway 192.168.1.1
      wpa-ssid namn_på_nätverket
      wpa-psk lösenord_för_nätverket
      dns-nameservers 8.8.8.8 192.168.1.1  

_notera att google's nameserver används_
