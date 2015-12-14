Ansluta en Ubuntu server till eduroam med WiFi

För att ansluta till eduroam via WiFi från din ubuntu server krävs lite konfiguration. Följande guide gäller uppkoppling mot eduroam på BTH (Blekinge tekniska högskola).

Se till att ditt WiFi-kort eller WiFi-dongel fungerar med drivrutin etc.
Ta reda på namnet på ditt nätverkskort (dongel). Vanligen: wlan0
Ladda ner certifikatet för att få ansluta till BTHs nätverk och spara det på lämplig plats.

sudo wget -O /etc/ssl/certs/AddTrust-External-CA-Root.cer https://raw.githubusercontent.com/dite-bth/hackday/master/151211/resources/AddTrust-External-CA-Root.cer

Redigera filen /etc/wpa_supplicant/wpa_supplicant.conf med en texteditor (exemplen nedan använder vim och nano). Tänk på att du måste göra detta med "super user"-rättigheter (sudo)

sudo vi /etc/wpa_supplicant/wpa_supplicant.conf
eller
sudo nano /etc/wpa_supplicant/wpa_supplicant.conf

Skriv följande i filen

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

Redigera filen /etc/networks/interfaces och skriv in följande i filen (om det inte redan står). Innan du redigerar bör du ta reda på vad ditt nätverkskort heter. Nedan används wlan0, men det kan heta något annat t ex wlp1s0. För att ta reda på vad ditt heter kan du kolla output från ifconfig eller iwconfig. Om det heter något annat, byt ut wlan0 till aktuellt namn.

auto wlan0
allow-hotplug wlan0
iface wlan0 inet manual
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf

OBS!
Om ovanstående inte fungerar kan du testa att byta ut till att få ett ip via dhcp;

auto wlan0  
allow-hotplug wlan0  
iface wlan0 inet dhcp  
wpa-conf /etc/wpa_supplicant/wpa_supplicant.conf  
Kör sedan;
$ sudo ifdown wlan0 && ifup wlan0
testa sedan t ex genom att pinga;
$ ping -c3 www.google.com

6. Till sist - testa att reboota för att kolla så att allt funkar.   
Med kommandot ifconfig kan du kolla om ditt nätverkskort har fått en IP-adress av DHCP-servern. Leta efter "wlan0" (eller vad ditt nätverkskort heter) i outputen.

`sudo ifconfig`