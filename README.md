# Raspberry Pico HSM
This project aims to transform a Raspberry Pi Pico or ESP32 microcontroller into a Hardware Security Module (HSM). The modified Pico or ESP32 board will be capable of generating and storing private keys, performing AES encryption or decryption, and signing data without exposing the private key. Specifically, the private key remains securely on the board and cannot be retrieved since it is encrypted within the flash memory.

## Capabilities
### > Key generation and encrypted storage
Private and secret keys are secured using a master AES 256 key (MKEK). The MKEK is encrypted with a hashed and salted version of the PIN.
**No private/secret keys, DKEK or PIN are stored in plain text ever. Never.**

### > RSA Key Generation (1024 to 4096 Bits)
RSA key generation is supported for 1024, 2048, 3072, and 4096 bits. Private keys never leave the device.

### > ECDSA Key Generation (192 to 521 Bits)
ECDSA key generation supports various curves from 192 to 521 bits.

### > ECC Curves
Supported ECC curves include secp192r1, secp256r1, secp384r1, secp521r1, brainpoolP256r1, brainpoolP384r1, brainpoolP512r1, secp192k1 (insecure), secp256k1, Curve25519, and Curve448.

### > SHA Digests
ECDSA and RSA signatures can be combined with SHA-1, SHA-224, SHA-256, SHA-384, and SHA-512 digests.

### > Multiple RSA Signature Algorithms
Supported RSA signature algorithms include RSA-PSS, RSA-PKCS, and raw RSA signatures.

### > ECDSA Signatures
ECDSA signatures can be raw or pre-hashed.

### > ECDH Key Derivation
Supports the ECDH algorithm for calculating shared secrets.

### > EC Private Key Derivation
Allows ECDSA key derivation.

### > RSA Decryption
Supports RSA-OEP and RSA-X.509 decryption.

### > AES Key Generation
Supports AES key generation with keys of 128, 192, and 256 bits.

### > AES-CBC Encryption/Decryption
Performs AES-CBC encryption and decryption.

### > Advanced AES Modes
Supports AES encryption and decryption in ECB, CBC, CFB, OFB, XTS, CTR, GCM, and CCM modes, with customizable IV/nonce and additional authenticated data (AAD).[^4]

### > AES Key Generation (128, 192, 256, 512 Bits)
Supports AES key generation up to 512 bits, useful for AES XTS where two 256-bit keys are concatenated.

### > CMAC
Supports AES-CMAC authentication.[^1]

### > AES Secret Key Derivation
Supports AES secret key derivation.[^1]

### > PIN Authorization
Private and secret keys require prior PIN authentication. Supports alphanumeric PINs.

### > PKCS11 Compliant Interface
Interfacing with the PKCS11 standard is supported.

### > Hardware Random Number Generator (HRNG)
Contains an HRNG designed for maximum entropy.

### > Device Key Encryption Key (DKEK) Shares
Supports importing DKEK shares to wrap, unwrap, and encrypt keys.

### > DKEK n-of-m Threshold Scheme
Supports an n-of-m threshold scheme to prevent outages when a DKEK custodian is unavailable.

### > USB/CCID Support
Full USB CCID stack for communication with the host via OpenSC and PCSC, allowing the use of frontend applications like OpenSSL via the PKCS11 module.

### > Extended APDU Support
Supports extended APDU packets, allowing up to 65535 bytes.

### > CV Certificates
Handles CVC certificates and requests to minimize internal certificate storage.

### > Attestation
Each generated key is attached to a certificate signed by an external PKI, ensuring the key was generated by the specific device.

### > Import External Keys and Certificates
Allows importing private keys and certificates via WKY or PKCS#12 files.[^2][^3]

### > Transport PIN
Allows a transport PIN for provisioning, ensuring the device has not been tampered with during transportation.[^2]

### > Press-to-Confirm Button
Uses the BOOTSEL button to confirm operations with private/secret keys, providing a 15-second window to confirm the operation to protect against unauthorized use.

### > Store and Retrieve Binary Data
Allows the storage of arbitrary binary data files.

### > Real-Time Clock (RTC)
Includes an RTC with external date and time setting and retrieval.

### > Secure Messaging
Supports secure channels to encrypt data packets between the host and device, preventing man-in-the-middle attacks.

### > Session PIN
A specific session PIN can be set during session opening to avoid systematic PIN usage.

### > PKI CVCert Remote Issuing for Secure Messaging
Secure channel messages are secured with a certificate issued by an external PKI.

### > Multiple Key Domains
Supports separate key domains protected by independent DKEKs, allowing different keys in different domains.

### > Key Usage Counter
Tracks and limits the usage of private/secret keys, disabling keys once their usage counter reaches zero.

### > Public Key Authentication (PKA)
Supports PKA for enhanced security, requiring a secondary device for authentication using a challenge-response mechanism.

### > Secure Lock
Adds an extra layer of security by locking the Pico HSM to a specific computer using a private key.

### > ChaCha20-Poly1305
Supports the ChaCha20-Poly1305 encryption algorithm for secure data encryption.[^4]

### > X25519 and X448
Supports DH X25519 and X448 for key agreement, though these cannot be used for signing.

### > Key Derivation Functions
Supports HKDF, PBKDF2, and X963-KDF for symmetric key derivation.

### > HMAC
Supports HMAC generation with SHA digest algorithms.

### > CMAC
Supports CMAC with AES for keys of 128, 192, and 256 bits.

### > XKEK
Supports an advanced key sharing scheme (XKEK) for securely wrapping and unwrapping keys within authorized domains.

### > Master Key Encryption Key (MKEK)
Uses an MKEK to securely store all keys, encrypted with an ephemeral key derived from the hashed PIN.

### > Hierarchical Deterministic Key Generation
Supports BIP32 for asymmetric key derivation and SLIP10 for symmetric key derivation, enabling crypto wallet deployment with infinite key generation. Supports NIST 256 and Koblitz 256 curves for master key generation.[^4]

[^1]: PKCS11 modules (`pkcs11-tool` and `sc-tool`) do not support CMAC and key derivation. It must be processed through raw APDU command (`opensc-tool -s`).
[^2]: Available via SCS3 tool. See [SCS3](/doc/scs3.md "SCS3") for more information.
[^3]: Imports are available only if the Pico HSM is previously initialized with a DKEK and DKEK shares are available during the import process.
[^4]: Available by using PicoHSM python tool.

### > ESP32-S3 support
Pico HSM also supports ESP32-S3 boards, which add secure storage, flash encryption and secure boot.

### > Dynamic VID/PID
Supports setting VID & PID on-the-fly. Use `pico-hsm-tool.py` for specify VID/PID values and reboot the device.

### > Rescue Pico HSM Tool
Pico HSM Tool implements a new CCID stack to rescue the Pico HSM in case it has wrong VID/PID values and it is not recognized by the OS.

## Security considerations
All secret keys (both asymmetric and symmetric) are encrypted and stored in the flash memory of the Raspberry Pico. The DKEK, a 256-bit AES key, is used to protect these private and secret keys. Keys are only held in RAM during signature and decryption operations, and are loaded and cleared each time to avoid potential security vulnerabilities.

The DKEK itself is encrypted using a doubly salted and hashed PIN, and the PIN is hashed in memory during sessions. This ensures that the PIN is never stored in plain text, either in flash memory or in RAM. However, if no secure channel is used, the PIN is transmitted in plain text from the host to the HSM.

In the event that the Pico is stolen, the private and secret key contents cannot be accessed without the PIN, even if the flash memory is dumped.

## Download
**If you own an ESP32-S3 board, go to [ESP32 support](https://www.picokeys.com/esp32-support/) for further information.**

Please, go to the Release page and download the UF2 file for your board.

Note that UF2 files are shiped with a dummy VID/PID to avoid license issues (FEFF:FCFD). If you plan to use it with OpenSC or similar tools, you should modify Info.plist of CCID driver to add these VID/PID or use the [Pico Patcher tool](https://www.picokeys.com/pico-patcher/).

Alternatively you can use the legacy VID/PID patcher as follows:
`./patch_vidpid.sh VID:PID input_hsm_file.uf2 output_hsm_file.uf2`

You can use whatever VID/PID (i.e., 234b:0000 from FISJ), but remember that you are not authorized to distribute the binary with a VID/PID that you do not own.

Note that the pure-browser option [Pico Patcher tool](https://www.picokeys.com/pico-patcher/) is the most recommended.

## Build
Before building, ensure you have installed the toolchain for the Pico and the Pico SDK is properly located in your drive.

```
git clone https://github.com/polhenarejos/pico-hsm
git submodule update --init --recursive
cd pico-hsm
mkdir build
cd build
PICO_SDK_PATH=/path/to/pico-sdk cmake .. -DPICO_BOARD=board_type -DUSB_VID=0x1234 -DUSB_PID=0x5678
make
```
Note that `PICO_BOARD`, `USB_VID` and `USB_PID` are optional. If not provided, `pico` board and VID/PID `FEFF:FCFD` will be used.

Additionally, you can pass the `VIDPID=value` parameter to build the firmware with a known VID/PID. The supported values are:

- `NitroHSM`
- `NitroFIDO2`
- `NitroStart`
- `NitroPro`
- `Nitro3`
- `Yubikey5`
- `YubikeyNeo`
- `YubiHSM`
- `Gnuk`
- `GnuPG`

After running `make`, the binary file `pico_hsm.uf2` will be generated. To load this onto your Pico board:

1. Put the Pico board into loading mode by holding the `BOOTSEL` button while plugging it in.
2. Copy the `pico_hsm.uf2` file to the new USB mass storage device that appears.
3. Once the file is copied, the Pico mass storage device will automatically disconnect, and the Pico board will reset with the new firmware.
4. A blinking LED will indicate that the device is ready to work.

### Docker
Independent from your Linux distribution or when using another OS that supports Docker, you could build a specific pico-hsm version in a Linux container.

```
sudo docker build \
    --build-arg VERSION_PICO_SDK=1.5.0 \
    --build-arg VERSION_MAJOR=3 \
    --build-arg VERSION_MINOR=4 \
    --build-arg PICO_BOARD=waveshare_rp2040_zero \
    --build-arg USB_VID=0xfeff \
    --build-arg USB_PID=0xfcfd \
    -t pico-hsm-builder .

sudo docker run \
    --name mybuild \
    -it pico-hsm-builder \
    ls -l /home/builduser/pico-hsm/build_release/pico_hsm.uf2

sudo docker cp mybuild:/home/builduser/pico-hsm/build_release/pico_hsm.uf2 .

sudo docker rm mybuild
```

## Usage
The firmware uploaded to the Pico contains a reader and a virtual smart card, similar to having a physical reader with an inserted SIM card. We recommend using [OpenSC](http://github.com/opensc/opensc/ "OpenSC") to communicate with the reader. If OpenSC is not installed, you can download and build it or install the binaries for your system.

To ensure that the Pico is detected as an HSM, use the following command:
```sh
opensc-tool -an
```
It should return a text similar to:
```sh
Using reader with a card: Free Software Initiative of Japan Gnuk
3b:fe:18:00:00:81:31:fe:45:80:31:81:54:48:53:4d:31:73:80:21:40:81:07:fa
SmartCard-HSM
```
The name of the reader may vary if you modified the VID/PID.

For further details and operations, refer to the following documentation:

- Initialization and Asymmetric Operations [doc/usage.md](/doc/usage.md)
- Signing and Verification Operations [doc/sign-verify.md](/doc/sign-verify.md)
- Asymmetric Encryption and Decryption [doc/asymmetric-ciphering.md](/doc/asymmetric-ciphering.md)
- Backup, Restore, and DKEK Share Management [doc/backup-and-restore.md](/doc/backup-and-restore.md)
- AES Key Generation, Encryption, and Decryption [doc/aes.md](/doc/aes.md)
- 4096 Bits RSA Support [doc/scs3.md](/doc/scs3.md)
- Storing and Retrieving Arbitrary Data [doc/store_data.md](/doc/store_data.md)
- Extra Options (e.g., set/get real datetime, enable/disable press-to-confirm button [doc/extra_command.md](/doc/extra_command.md)
- Public Key Authentication [doc/public_key_authentication.md](/doc/public_key_authentication.md)

## Operation time
### Keypair generation
Generating EC keys is almost instant. RSA keypair generation takes some time, specially for `3072` and `4096` bits.

| RSA key length (bits) | Average time (seconds) |
| :---: | :---: |
| 1024 | 16 |
| 2048 | 124 |
| 3072 | 600 |
| 4096 | ~1000 |

### Signature and decrypt
| RSA key length (bits) | Average time (seconds) |
| :---: | :---: |
| 1024 | 1 |
| 2048 | 3 |
| 3072 | 7 |
| 4096 | 15 |

## Press-to-confirm button
The Raspberry Pico includes a BOOTSEL button used for loading firmware initially. Once the Pico HSM firmware is running, this button can be repurposed for additional functionalities. Specifically, the Pico HSM utilizes this button to confirm private and secret operations, a feature that is optional but highly recommended for enhanced security.

When enabled, each time a private or secret key operation is initiated, the Pico HSM enters a waiting state where it awaits user confirmation by pressing the BOOTSEL button. During this waiting period, the Pico HSM's LED remains mostly illuminated but blinks off briefly every second, signaling to the user to press the button for confirmation. If no action is taken, the Pico HSM will continue to wait indefinitely. This operation mode includes periodic timeout commands sent to the host to prevent the session from timing out prematurely.

This feature adds an additional layer of security by requiring physical user intervention for sensitive operations such as signing or decrypting data. It mitigates risks associated with unauthorized applications or scripts using the Pico HSM without user awareness. However, it is not recommended for server environments or other automated settings where physical access to press the button may not be practical.

For more details on configuring and using this feature, refer to the [doc/extra_command.md](/doc/extra_command.md) document.

## Led blink
Pico HSM uses the led to indicate the current status. Four states are available:

### Press to confirm
The Led is almost on all the time. It goes off for 100 miliseconds every second.

![Press to confirm](https://user-images.githubusercontent.com/55573252/162008917-6a730eac-396c-44cc-890e-802294be30a3.gif)

### Idle mode
In idle mode, the Pico HSM goes to sleep. It waits for a command and it is awaken by the driver. The Led is almost off all the time. It goes on for 500 milliseconds every second.

![Idle mode](https://user-images.githubusercontent.com/55573252/162008980-d5a5caad-072e-400c-98e3-2c606b4b2af9.gif)

### Active mode
In active mode, the Pico HSM is awaken and ready to receive a command. It blinks four times in a second.

![Active](https://user-images.githubusercontent.com/55573252/162008997-1ea8cd7e-5384-4893-9dcb-b473153fc375.gif)

### Processing
While processing, the Pico HSM is busy and cannot receive additional commands until the current is processed. In this state, the Led blinks 20 times in a second.

![Processing](https://user-images.githubusercontent.com/55573252/162009007-df45111e-2473-4a92-97c5-15c3cd19babd.gif)

## Driver

The Pico HSM uses either the `sc-hsm` driver from [OpenSC](https://github.com/OpenSC/OpenSC/) or the `sc-hsm-embedded` driver from [CardContact](https://github.com/CardContact/sc-hsm-embedded/) to interface with external applications. These drivers employ the standardized PKCS#11 interface, making it compatible with various cryptographic engines that support PKCS#11, such as OpenSSL, P11 library, or pkcs11-tool.

Internally, the Pico HSM organizes and manages its data using the PKCS#15 structure, which includes elements like PINs, private keys, and certificates. Commands can be issued to interact with these stored elements using tools such as `pkcs15-tool`. For example, `pkcs15-tool -D` lists all elements stored within the Pico HSM.

Communication with the Pico HSM follows the same protocols and methods used with other smart cards, such as OpenPGP cards or similar devices.

For advanced usage scenarios, refer to the documentation and examples provided. Additionally, the Pico HSM supports the SCS3 tool for more sophisticated operations and includes features like multiple key domains. For detailed information on SCS3 usage, refer to [SCS3 documentation](/doc/scs3.md).

### Important
OpenSC relies on PCSC driver, which reads a list (`Info.plist`) that contains a pair of VID/PID of supported readers. In order to be detectable, you have several options:
- Use `pico-hsm-tool.py` to modify VID/PID on-the-fly.
- Use the online [Pico Patcher tool](https://www.picokeys.com/pico-patcher/).
- Patch the UF2 binary (if you just downloaded from the [Release section](https://github.com/polhenarejos/pico-hsm/releases "Release section"))
- Build and configure the project with the proper VID/PID with `USB_VID` and `USB_PID` parameters in `CMake` (see [Build section](#build "Build section")). Note that you cannot distribute the patched/compiled binary if you do not own the VID/PID or have an explicit authorization.

## Credits
Pico HSM uses the following libraries or portion of code:
- mbedTLS for cryptographic operations.
- TinyUSB for low level USB procedures.
