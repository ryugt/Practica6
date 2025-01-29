#Introducción
El problema presentado consiste en desencriptar una carpeta cifrada que contiene credenciales de acceso a varios bancos rusos, las cuales son utilizadas por un grupo de cibercriminales para transferir dinero obtenido de rescates de ransomware. La carpeta cifrada está protegida por una clave de cifrado, la cual a su vez está cifrada con la clave pública de un certificado. En este escenario, se cuenta con la clave pública y privada del certificado, pero se desconoce la contraseña necesaria para acceder a la clave privada.
#Objetivo
El objetivo es obtener el login y la contraseña de acceso al banco Gazprombank, para lo cual se deben seguir una serie de pasos que permitan desencriptar la carpeta cifrada y acceder a las credenciales.
Archivos Disponibles
Para resolver este problema, se dispone de la siguiente información y archivos contenidos en Practica6.zip:
-	El mensaje de correo cifrado está en el archivo mail.msg
-	Los certificados digitales con las claves públicas de los cinco destinatarios del mensaje están en el archivo bundle.pem
-	El certificado digital con las claves pública y privada están en el archivo EvilHacker.pfx
-	La carpeta cifrada con todos los archivos, incluido el que contiene las credenciales de acceso a los bancos, se encuentra en el archivo EncryptedFolder.xml
#Metodología
Para llevar a cabo la resolución del problema, se seguirán los siguientes pasos detallados:
##1.	Desencriptar el Mensaje de Correo
El primer paso consiste en guardar los archivos en la carpeta: 0_Material_Original
Posterior se comienza a desencriptar el mensaje de correo cifrado (mail.msg) que contiene la contraseña de acceso al certificado con las claves pública y privada. Para ello se realizan los siguientes pasos:
###1.1.	Revisión archivo mail.msg
Para realizar este paso se procede a revisar el archivo con openssl
El comando usado es:
openssl smime -pk7out -in 0_Material_Original/mail.msg | openssl pkcs7 -print -text -noout
Con el cual se puede ver las enc_key asignada a cada uno de los 5 destinatarios del mensaje de correo.
Se identifica 2 campos importantes:
-	serial
El cual permitirá realizar la relación con los certificados públicos usados
-	enc_key
El cual es la clave de encriptación usada para encriptar el mensaje, la cual esta encriptada con la clave publica del usuario
 
Salida del email mail.msg
Adicional se identifica el mensaje cifrado (el mensaje encriptado enc_data con su respectivo iv [OCTET STRING] )
 
Mensaje encriptado en mail.msg
###1.2.	Revisión bundle.pem
Se realiza la separación del archivo en 5 archivos individuales, las mismas en la carpeta: Cer_Pub_atack
-	kpub1.pem
-	kpub2.pem
-	kpub3.pem
-	kpub4.pem
-	kpub5.pem
Se revisa el contenido para revisar el módulo N y el valor exponente e con el comando:
openssl x509 -in Cer_Pub_atack/kpub<id>.pem -nouut -text
Se ve que los certificados fueron realizados con RSA de 1024 bits, y que su valor exponente en las 5 es de valor 5 (e = 5).
 
Salida del certificado kpub1.pem
Con la confirmación que el valor del exponente e es 5 en los 5 certificados, y que el valor es bajo, más la pista dada, se procede a realizar el ataque Teorema del Resto Chino (CRT), por lo cual se procede a sacar los módulos y convertirlos en valor decimal:
-	156347688102726454823889416472453769630029897853512166254687777486075063031423183676252895347432482500241946689013358855470486673169586930678111085911649116120388122215781545716642791931518186610854646243628090444213625898861906056515185507685253458803609826204999300235298709231490589094202247797312708961907
-	117848142580767301479140610710350191400363802862119303486145436424386403651945506124604731078881678785302566535894067779696338606654375587912054236667905745167180779541914821605115675550265348648694791854824634994884784098347860611336863256025292645012883471253871196440185889342884709931056940458285831881759
-	160840782788886869832552217679031227306145298760248558604239324564642932511809556758723771493024027652356917672632695677849531127766622621206334981013367305231061396447089403731525465581041597768882342165534924271303668908582936231000186385983869381704613464423017749139940223866684432378537046831053915159771
-	109572690095833540015177630428364938495950572655072149382179286384922852732906777196360212411251388705330226247771356213394179053422433827490965557757599881601396359035094633961986603390874697313171100402004246306710761908700692386929283300038933941891301836103671930246156416077878801122403583579122661211123
-	141701375513385166436096856851870377569757196618323301515474561851963944201553892806385634685979719434134626892867514566869672175318054795127061123136343151046388750768853585184965464447778660441245369717291925456813496616014316315880303824874971158040757712062942215019738814733157445385148149570670759906769
Del paso 1.1, se realiza la comparación del Serial number para hacer el match con el Serial en cada certificado público, luego se obtiene las 5 claves encriptadas (enc_key) 
-	"93B2F23D212AD4FE2B8AF2CF095ACCF51D40A49E228A46DB124EE48951560755BB0947E49709FD7C323721B9C75A1A420BAAFD454E13C4C972A7AA6A6F30924E688CE9B661B104E6CF2B06CD38EEFDD75F1DAFA7BF37E41D1CED96B1D6A756CD8DD15E1B481266EBEC5AB228562825E181513F35D1083D776CC805F748A280DE" 
-	"629A73AB9C2E984540C0E6E51F9B511102188360F1189703E14C6256AE337FEDC99586B270E6CAAF806AC834260E3C6ACBCD20BB4BC7AAACDA740A0E3063AFBD475C6DEAC370303C6C6FDCC7CC0E4F0EC2032D0EAD1A84A7955AA6E0299DD57043F6B056D00796F60F3249B2824247CCA68D70AA8F6E0693AF23AD8EDD95D627" 
-	"78DF06A6E6A828630DF0E51E67FC0F4F7095BBE0B03FD1790901C2D167ADD01E08452A2E7F5E41C4120C18B0CDE3A23C38407BAF3452C23CECF6D2B63ED677A92931E435FA9F52D7762F31125AEEFEC7317DFC6D3B7124278825A54EF09CD1552A9FC22EE41D77FDF01FDF595DB808E55FD73EE62C11F83868A3942459276707" 
-	"0DEE7FEACC216E79AF3E5A33014655912EF8A6B3B95F991E5440117C05E43AB23786CD3FC8960D4B33308A1E83050552D02160DEAB7495F2323ACF8F8BF0EB917D7EC848C07CA13F7DA00C7D70C7ABE53FF58F36D000F62954E029E5DEEA2BD376317CF67E31DFDD1FE8D8FDE422826BB0E6430C9653D5CE30FC1FBC127BB3EE"
-	"B68697D66DD835BAC0AFD070D47B7C04A31782FFFC24B312EAC3EE2BB9DD30B096CE73CDCFD1946689387A5C8A674CEFF98AC5D3E961DC1BC1A62D3FA91DBC891A350A7F44F24EAB9DA91A7AAC4198A18BC76F487C1DC5E4C99BF38434BE9C52CDB28B83E587E1D88A991260571D0923E2C30DEFA497ABD76D3B7A28166A7D80"
Con los valores obtenidos, se usa el script en Python que permitirá realizar el cálculo de la raíz quíntupla del CRT (CRT.py), lo que nos da el siguiente resultado:
 
Salida de cálculo del CRT
Siendo el valor de K (desencriptado)
•	1d680dd0af91a59b6b3c7bdfff6c9479a85ab8c6c1ac320a0577edcb55f215ae
###1.3.	Desencriptación mail.msg
Se realiza la separación del enc_data con los valores del iv obtenidos y el valor de K anteriormente obtenidos; se guarda el valor de enc_data en el documento msg.enc, el cual se encuentra guardado: msg_enc_dec
Se usa el comando:
openssl enc -d -aes-256-cbc -in msg_enc_dec/mensaje.enc -K 1d680dd0af91a59b6b3c7bdfff6c9479a85ab8c6c1ac320a0577edcb55f215ae -iv B1A88E22D5583059CC2A1FA1FB596FBB -out msg_enc_dec/msg.txt
El cual nos da el mensaje desencriptado y ya se puede ver la clave del archivo pfx
 
Mensaje desencriptado
El contenido del mensaje es:
-	ImperatorskayaRossiya
##2.	Acceder a la Clave Privada del Certificado
Utilizando la contraseña obtenida en el paso anterior, se procederá a abrir el archivo EvilHacker.pfx y acceder a la clave privada del certificado. 
Para esto se usa el comando:
openssl pkcs12 -in 0_Material_Original/EvilHacker.pfx -nocerts -out Files_enc_dec/Kpriv_eh.pem
El cual nos permite obtener la clave privada y guardarlo en el archivo Kpriv_eh.pem 
openssl pkcs12 -in 0_Material_Original/EvilHacker.pfx -nokeys -clcerts -out Files_enc_dec/Kpub_eh.pem
El cual nos permite obtener la clave publica y guardarlo en el archivo Kpriv_eh.pem 
Los archivos se guardan en: Files_enc_dec
Se realiza la validación que la clave publica es la usada para encriptar los Secret_key del archivo EncryptedFolder.xml, por lo que se abre el archivo y se lo separa en archivos individuales.
###2.1.	Separación EncryptedFolder.xml
 
revisión del archivo XML
Se realiza la separación del archivo en 10 archivos individuales, las mismas en la carpeta: Files_enc_dec
Los archivos son:
La clave publica:
-	Kpub.pem
Los documentos encriptados (en formato binario):
-	F1.enc
-	F2.enc
-	F3.enc
Las claves de encriptación que están encriptadas con Kpub.pem (en formato binario):
-	KF1.enc
-	KF2.enc
-	KF3.enc
Los IV de cada archivo (en formato Hexadecimal):
-	IVF1.hex
-	IVF2.hex
-	IVF3.hex
Se realiza la comparación de ambos certificados: Kpub_eh.pem y Kpub.pem

##3.	Desencriptar los archivos
###3.1.	Desencriptación de KF con Krpiv_eh.pem
Se realiza la desencriptación con el siguiente comando:
openssl rsautl -decrypt -inkey Files_enc_dec/kpriv_eh.pem -in Files_enc_dec/KF<id>.enc -out Files_enc_dec/KF<id>.dec
El cual nos genera los siguientes archivos:
-	KF1.dec
-	KF2.dec
-	KF3.dec
Cuyos valores son (en hexadecimal):
-	E4D7514A30BCD2BC18ADAB61207FFA94
-	E00EEC7AA64A107BEA38965FBEA2AA05
-	0E4ACBF031C531F4EEC0BE27CCDECE1A
###3.2.	Desencriptación de F*.enc con KF e IV
Con los valores desencriptados de las claves, se procede a realizar la desencriptación de los archivos encriptados con AES-128, con el comando:
openssl aes-128-cbc -d -in Files_enc_dec/F*.enc -out Files_enc_dec/F*.dec -K Files_enc_dec/<KF*> -iv Files_enc_dec/<IVF*>
Cuyos valores de iv (en hexadecimal):
-	A35E980B835764E2B1E271C092461A85
-	0A2E359B00AE151B80CC477DDEA19DB4
-	51655D2F6B5EA74FE36E1EB34C37121F
Genera los siguientes archivos:
-	F1.dec
-	F2.dec
-	F3.dec
##4.	Acceder a las Credenciales de Gazprombank
Finalmente, con los archivos desencriptados se ve los contenidos (alamacenados en la carpeta: Files_enc_dec)
 
Lo solicitado por el problema se encuentra en el archivo encriptado numero 3 cuyo resultado es:
Gazprombank
login: gonzalo.alvarez
pwd: MH6KNu;D'm4p/"#}=X+9k

#Conclusión
El proceso descrito anteriormente permite la obtención de las credenciales de acceso al banco Gazprombank mediante una serie de pasos que implican la desencriptación de mensajes de correo y archivos cifrados. 
Se uso las herramientas indicadas (openssl y cryptool) y solo se creo un sript en Python para realizar el cálculo del CRT

