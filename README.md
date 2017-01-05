# LowCDC-Win10x64

This repo contains almost everything (except for the usbser.sys driver because of Microsoft Software License Terms) that is needed to create a lowcdc.sys driver package that can be installed on the 64-bit version of Windows 10. lowcdc.sys is published unchanged, the source code is available on the author's site (see the Credits section).

*I'm not very proficient in English, please consider helping with the translation. You can send a pull request, participate in [the discussion](https://github.com/protaskin/LowCDC-Win10x64/issues/3) or send me an email.*

## Why does not the lowcdc.sys driver install/work on Windows 10?

1. The lowcdc.inf installation script does not contain necessary sections (SourceDisksNames, SourceDisksFiles), the driver package does not contain a signed catalog file.

2. **[usbser.sys has been completely re-written in Windows 10](https://blogs.msdn.microsoft.com/usbcoreblog/2015/07/29/what-is-new-with-serial-in-windows-10/) and cannot be used with the current version of the lowcdc.sys.**

3. [Beginning with the release of Windows 10, all new Windows 10 kernel mode drivers must be submitted to and digitally signed by the Windows Hardware Developer Center Dashboard portal](https://blogs.msdn.microsoft.com/windows_hardware_certification/2015/04/01/driver-signing-changes-in-windows-10/).

## Getting Windows and the driver package ready

1. Find `usbser.sys` included in the 64-bit version of Windows 7. The file is located in the `\Sources\install.wim\Windows\System32\DriverStore\FileRepository\mdmcpq.inf_amd64_neutral_fbc4a14a6a13d0c8\` folder on the installation disk of Windows 7 with integrated SP1. The version of the driver I use is 6.1.7610.17514. Copy the file to the driver package's folder and rename it to `usbser61.sys` to avoid replacement of the Windows 10 driver.

2. Install [Windows 10 SDK](https://developer.microsoft.com/en-us/windows/downloads/windows-10-sdk) and [Windows Driver Kit (WDK) 10](https://developer.microsoft.com/en-us/windows/hardware/windows-driver-kit). Make sure that the `Inf2Cat` program is located in the `\Program Files (x86)\Windows Kits\10\Bin\x86\` folder, the programs `MakeCert`, `CertMgr`, `SignTool` are located in the `\Program Files (x86)\Windows Kits\10\Bin\x64\` folder.

3. [Enable the TESTSIGNING boot configuration option](https://msdn.microsoft.com/windows/hardware/drivers/install/the-testsigning-boot-configuration-option), restart the computer for the change to take effect. When the option for test-signing is enabled Windows displays a watermark with the text "Test Mode", the version and build of Windows in the lower right-hand corner of the desktop. **Be aware using Windows with the TESTSIGNING boot configuration option, Windows will load any type of test-signed kernel-mode code.**

4. [Create a catalog file for the driver package](https://msdn.microsoft.com/windows/hardware/drivers/install/creating-a-catalog-file-for-a-pnp-driver-package).

5. [Create a MakeCert test certificate](https://msdn.microsoft.com/windows/hardware/drivers/install/makecert-test-certificate), [install the test certificate to corresponding certificate stores](https://msdn.microsoft.com/windows/hardware/drivers/install/using-certmgr-to-install-test-certificates-on-a-test-computer), [test-sign the driver package's catalog file](https://msdn.microsoft.com/windows/hardware/drivers/install/test-signing-a-driver-package-s-catalog-file).

## Using createcat.bat

createcat.bat is a script that generates a test-signed catalog file for the driver package (performs the actions 4 and 5 from the list above).

The script does not need any configuration and ready for use. However you can change the name of a certificate (the `CertName` variable) or use an installed certificate (change the `CertName` variable, set `CreateCert=0`).

1. Run `createcat.bat` with the administrative permissions (it is not necessary to run the script in the command prompt).

2. Examine the output. The text below is the result of a successful execution.

<pre>
D:\LowCDC-Win10x64>createcat.bat

D:\LowCDC-Win10x64>cd /D C:\Program Files (x86)\Windows Kits\10\Bin\x86

C:\Program Files (x86)\Windows Kits\10\bin\x86>Inf2Cat /driver:"D:\LowCDC-Win10x64" /os:10_X64
.................................
Signability test complete.

Errors:
None

Warnings:
None

<b>Catalog generation complete.
D:\LowCDC-Win10x64\lowcdc.cat</b>

C:\Program Files (x86)\Windows Kits\10\bin\x86>cd ..\x64

C:\Program Files (x86)\Windows Kits\10\bin\x64>if 1 EQU 1 (if not exist "D:\LowCDC-Win10x64\certcopy.cer" (
MakeCert -r -pe -ss CA -n "CN=Artyom Protaskin CA" "D:\LowCDC-Win10x64\certcopy.cer"
 CertMgr /add "D:\LowCDC-Win10x64\certcopy.cer" /s /r localMachine root
 CertMgr /add "D:\LowCDC-Win10x64\certcopy.cer" /s /r localMachine trustedpublisher
) )
<b>Succeeded</b>
<b>CertMgr Succeeded</b>
<b>CertMgr Succeeded</b>

C:\Program Files (x86)\Windows Kits\10\bin\x64>SignTool sign /v /s CA /n "Artyom Protaskin CA" /t http://timestamp.verisign.com/scripts/timstamp.dll "D:\LowCDC-Win10x64\lowcdc.cat"
The following certificate was selected:
    Issued to: Artyom Protaskin CA
    Issued by: Artyom Protaskin CA
    Expires:   Sun Jan 01 02:59:59 2040
    SHA1 hash: 91E7FC5F14E0AEEE3211E438A01DA07BB9959091

Done Adding Additional Store
<b>Successfully signed: D:\LowCDC-Win10x64\lowcdc.cat</b>

Number of files successfully Signed: 1
Number of warnings: 0
Number of errors: 0

C:\Program Files (x86)\Windows Kits\10\bin\x64>SignTool verify /v /pa "D:\LowCDC-Win10x64\lowcdc.cat"

Verifying: D:\LowCDC-Win10x64\lowcdc.cat
Signature Index: 0 (Primary Signature)
Hash of file (sha1): 0CA6677A0F8F5F7E05B8331DC5C662E9B6ABC24C

Signing Certificate Chain:
    Issued to: Artyom Protaskin CA
    Issued by: Artyom Protaskin CA
    Expires:   Sun Jan 01 02:59:59 2040
    SHA1 hash: 91E7FC5F14E0AEEE3211E438A01DA07BB9959091

The signature is timestamped: Sat Jan 16 08:36:20 2016
Timestamp Verified by:
    Issued to: Thawte Timestamping CA
    Issued by: Thawte Timestamping CA
    Expires:   Fri Jan 01 02:59:59 2021
    SHA1 hash: BE36A4562FB2EE05DBB3D32323ADF445084ED656

        Issued to: Symantec Time Stamping Services CA - G2
        Issued by: Thawte Timestamping CA
        Expires:   Thu Dec 31 02:59:59 2020
        SHA1 hash: 6C07453FFDDA08B83707C09B82FB3D15F35336B1

            Issued to: Symantec Time Stamping Services Signer - G4
            Issued by: Symantec Time Stamping Services CA - G2
            Expires:   Wed Dec 30 02:59:59 2020
            SHA1 hash: 65439929B67973EB192D6FF243E6767ADF0834E4


<b>Successfully verified: D:\LowCDC-Win10x64\lowcdc.cat</b>

Number of files successfully Verified: 1
Number of warnings: 0
Number of errors: 0

C:\Program Files (x86)\Windows Kits\10\bin\x64>cd /D D:\LowCDC-Win10x64
</pre>

createcat.bat generates the test-signed catalog file `lowcdc.cat` and creates the `certcopy.cer` file that contains a copy of the certificate.

A common issue that can be encountered is the 0x800B0101 error.

```
SignTool Error: WinVerifyTrust returned error: 0x800B0101
        A required certificate is not within its validity period when verifying against the current system clock or the timestamp in the signed file.
```

Open `lowcdc.cat`, compare the singing time of the catalog file and the value of the certificate's 'valid from'. Adjust the system clock. Run `createcat.bat` again (do not delete `certcopy.cer`, the file is used to determine whether the certificate has been created and added to the certificate stores, otherwise set `CreateCert=0`).

## Screenshots

The installed driver in Device Manager.

![Device Manager](http://artyom.protaskin.ru/storage/lowcdc-win10x64/pictures/device-manager-screenshot.png)

Communication with the MicroProg programmer.

![The MicroProg programmer](http://artyom.protaskin.ru/storage/lowcdc-win10x64/pictures/microprog-screenshot.png)

[Communication with the AVRISP programmer](https://github.com/protaskin/LowCDC-Win10x64/issues/1#issuecomment-261777640).

![The AVPISP programmer](http://artyom.protaskin.ru/storage/lowcdc-win10x64/pictures/avrisp-screenshot.png)

## Credits

lowcdc.sys is developed by [Osamu Tamura](http://www.recursion.jp/prose/avrcdc/).
