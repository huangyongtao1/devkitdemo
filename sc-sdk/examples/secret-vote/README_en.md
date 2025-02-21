# secret-vote demo

English | [简体中文](README.md)

## Introduction
The secret-vote demo is the reference implementation of the anonymous voting system developed based on the Kunpeng BoostKit for Confidential Computing TrustZone Kit.

[Kunpeng BoostKit for Confidential Computing TrustZone Kit](https://www.hikunpeng.com/en/developer/boostkit/confidential-computing)

It mainly consists of four modules:
- Client application (CA). It is used to compile and generate libsecret-vote-ca.so to be invoked by web services.
- Trusted application (TA). It runs on a secure OS and uses the Global Platform (GP) interface to encrypt, decrypt, and store data.
- Service Web service, which is developed using the Django framework of Python, provides REST APIs. It invokes the libsecret-vote-ca.so file compiled by the CA to apply for public and private key pairs, and encrypt and decrypt voting data.
- Frontend of the UI, the voting UI developed using vue and element-ui, supports user login, logout, voting, and voting result display.

## Main Directory Description
- CA: CA source code
- TA: TA source code
- ui: Frontend source code. Users can use source code to build the source code package.
- service: Backend source code
- static: Frontend static file. It is a built static file and can be used directly.
- lib: Directory for storing the .so files generated during CA compilation. The web service loads the .so files generated by the CA from this directory.
- pubkey: Directory generated during user login and voting, which stores the root public key and user public key.
- build: Build script

## Voting Service Process
1. Initialization
    - Create 10 users.
    - Invoke the CA interface to apply for the root public and private key pair.
    - The CA invokes the GP interface of the TA to generate the RSA root public and private key pair.
    - The TA uses the secure storage API of the GP to save the root public and private key pair, and returns the root public key to the CA.
    - The CA uses the OpenSSL interface to store the root public key as a .pem file.
2. User Login
    - Check whether the user has applied for a public key. If not, apply for a public key.
    - Invoke the CA interface to apply for the user's public key and signature.
    - The CA invokes the GP interface of the TA to generate the RSA user public key pair.
    - The TA uses the secure storage API of the GP to store the user's public and private key pair.
    - The TA reads the root public and private key pair, signs the user's public key, and returns the user's public key and signature data to the CA.
    - The CA uses the OpenSSL interface to store the user's public key as a .pem file and returns the signature data to the web client.
    - The web client saves the signature data to the database.
3. Voting
    - The web client reads the user's public key and signature, and invokes the voting interface provided by the CA to vote.
    - The CA reads the root public key and uses the OpenSSL interface to verify the signature of the user's public key.
    - The CA randomly generates parameters (key, nonce, add) required for AES encryption.
    - Use the user public key to encrypt AES parameters.
    - Invoke the TA interface to decrypt and save the data.
    - The TA uses the user's public and private key pair to decrypt data.
    - After the decryption, the TA invokes the secure storage API of the GP to save the AES parameters.
    - The CA uses the AES parameters to encrypt the voting data and invokes the TA interface to decrypt the voting data.
    - The TA uses the AES parameters to decrypt the voting data and returns it to the TA.
    - The Web client obtains the voting result and saves it to the database.


## Dependencies
The CA and TA need to be compiled before use. Use OpenSSL for the CA. The web service requires Python and SQLite3. The software and earliest versions required for building and running are as follows:

| Software | Earliest Version |
| -------- | ---------------- |
| gcc      | 4.8.5            |
| cmake    | 2.8              |
| openssl  | 1.1.1            |
| Python   | 3.7              |
| sqlite3  | 3.8.3            |

1. Ensure that the confidential computing SDK is installed in the environment. You need to install both kunpeng-sc and kunpeng-sc-devel. [Download at Confidential Computing SDK.](https://mirrors.huaweicloud.com/kunpeng/archive/Kunpeng_SDK/itrustee/)
2. Run the `lsmod | grep tzdriver` command to check whether the tzdriver is properly loaded.
3. Run the `ps -ef | grep teecd` command to check whether the daemon process is started properly.

## Usage Guide

**This project code and deployment process are for reference only. Do not apply directly to the production environment.**

1. Obtain the code.

```
git clone https://github.com/kunpengcompute/devkitdemo.git
```

2. Go to the folder where the secret-vote demo is stored.

```
cd devkitdemo/sc-sdk/examples/secret-vote/
```

3. Change TA_UUID in CA/secret_vote_ca.h to the UUID used by the developer to apply for the certificate.

4. Create a Python virtual environment.

```
python3 -m venv venv
```

5. Activate the virtual environment.

```
source venv/bin/activate
```

6. Compile the CA.

```
bash build/build_ca.sh
```

7. Replace the manifest.txt file in the ./TA/ directory with the manifest.txt file used for applying for a developer certificate.

8. Change the developer private key and config file path in TA/config_cloud.ini.

9. Compile the TA.

**The TA verifies the executable program path of the CA. You need to change PYTHON3_PATH in TA/secret_vote_ta.h to the absolute path of Python 3 in the virtual environment.**

```
bash build/install_ta.sh
```

10. Install the dependencies required for running the DEMO.

```
cd service
pip install -r requirements.txt
```

11. Initialize the database table structure.

```
python3 manage.py makemigrations
python3 manage.py migrate
```

12. Initialize the data content.

**The TA verifies the executable program path of the CA. The absolute path of Python 3 must be used.**

```
/path/to/python3 manage.py init_db
```

**By default, 10 users user_00-user_09 are created in the database. The default password is 123456789.**

13. Run the DEMO.

**The TA verifies the executable program path of the CA. The absolute path of Python 3 must be used.**

```
/path/to/python3 manage.py runserver IP:PORT
```

Go to http://IP:PORT/static/index.html


## Building and Running the Frontend Code

1. Download third-party package.
```
npm install
```

2. Compile and run the development mode.
```
npm run serve
```

3. Compile and package the production mode. The package is stored in the dist folder of the ui by default.
```
npm run build
```
