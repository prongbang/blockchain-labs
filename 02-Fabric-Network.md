## 1.ดาวน์โหลด Docker Images ของ Fabric
```
$ cd ~/Hyperledger && curl -sSL https://goo.gl/byy2Qj | bash -s 1.0.5
```

รอจนกว่าจะดาวน์โหลดเสร็จ
```
===> Downloading platform binaries
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
 64 22.6M   64 14.5M    0     0   104k      0  0:03:41  0:02:22  0:01:19  113k
```

ดู Image ทั้งหมดโดยใช้คำสั่ง 
```
$ docker image
```
เมื่อเสร็จแล้วจะได้ Docker Image ประมาณนี้
```
===> List out hyperledger docker images
hyperledger/fabric-tools       latest              6a8993b718c8        2 months ago        1.33GB
hyperledger/fabric-tools       x86_64-1.0.5        6a8993b718c8        2 months ago        1.33GB
hyperledger/fabric-couchdb     latest              9a58db2d2723        2 months ago        1.5GB
hyperledger/fabric-couchdb     x86_64-1.0.5        9a58db2d2723        2 months ago        1.5GB
hyperledger/fabric-kafka       latest              b8c5172bb83c        2 months ago        1.29GB
hyperledger/fabric-kafka       x86_64-1.0.5        b8c5172bb83c        2 months ago        1.29GB
hyperledger/fabric-zookeeper   latest              68945f4613fc        2 months ago        1.32GB
hyperledger/fabric-zookeeper   x86_64-1.0.5        68945f4613fc        2 months ago        1.32GB
hyperledger/fabric-orderer     latest              368c78b6f03b        2 months ago        151MB
hyperledger/fabric-orderer     x86_64-1.0.5        368c78b6f03b        2 months ago        151MB
hyperledger/fabric-peer        latest              c2ab022f0bdb        2 months ago        154MB
hyperledger/fabric-peer        x86_64-1.0.5        c2ab022f0bdb        2 months ago        154MB
hyperledger/fabric-javaenv     latest              50890cc3f0cd        2 months ago        1.41GB
hyperledger/fabric-javaenv     x86_64-1.0.5        50890cc3f0cd        2 months ago        1.41GB
hyperledger/fabric-ccenv       latest              33feadb8f7a6        2 months ago        1.28GB
hyperledger/fabric-ccenv       x86_64-1.0.5        33feadb8f7a6        2 months ago        1.28GB
hyperledger/fabric-ca          latest              002c9089e464        2 months ago        238MB
hyperledger/fabric-ca          x86_64-1.0.5        002c9089e464        2 months ago        238MB
```

## ตั้งค่า Environment ของ ```Hyperledger/bin``` ที่เราทำการดาวน์โหลดมา 
[https://github.com/golang/go/wiki/SettingGOPATH](https://github.com/golang/go/wiki/SettingGOPATH)

ซึ่งผมใช้ zshrc จะเข้าไปที่
```
$ open ~/.zshrc
```
ถ้าใช้ bash จะเข้าไปที่
```
$ open ~/.bash_profile
```
แล้วเพิ่มบรรทัดนี้เข้าไป
```
export PATH=$HOME/Hyperledger/bin:$PATH
```

ทีนี้ลองดูว่าสามารถเรียกคำสั่ง cryptogen ได้หรือเปล่า
```
$ cryptogen
```
ถ้าได้แบบนี้ถือว่าผ่าน
```
➜  ~ cryptogen
usage: cryptogen [<flags>] <command> [<args> ...]

Utility for generating Hyperledger Fabric key material

Flags:
  --help  Show context-sensitive help (also try --help-long and --help-man).

Commands:
  help [<command>...]
    Show help.

  generate [<flags>]
    Generate key material

  showtemplate
    Show the default configuration template

  version
    Show version information

➜  ~ 
```

## Build your ```first network```
ให้เรา clone repository ตั้งต้นของ Hyperledger Fabric มาไว้ที่ Workspace ของเรา
```
$ git clone -b master https://github.com/hyperledger/fabric-samples.git
```
โดยจะมีไฟล์และ folder ดังนี้
```
LICENSE                  balance-transfer         chaincode-docker-devmode first-network
MAINTAINERS.md           basic-network            fabcar                   high-throughput
README.md                chaincode                fabric-ca                scripts
```
เข้าไปที่ first-network
```
$ cd first-network  && ls
```
จะเจอไฟล์ประมาณนี้
```
README.md                        configtx.yaml                    docker-compose-couch.yaml        org3-artifacts
base                             crypto-config.yaml               docker-compose-e2e-template.yaml scripts
byfn.sh                          docker-compose-cli.yaml          docker-compose-org3.yaml
channel-artifacts                docker-compose-couch-org3.yaml   eyfn.sh
```
> จากนั้นเราจะใช้ ```byfn.sh``` ซึ่งเป็น script ที่ช่วยรันคำสั่งต่าง ๆ ทั้งหมดให้เราในการสร้าง ลบ หรือรีเซตเน็ตเวิร์กของเรา ให้เรารัน
```
$ bash ./byfn.sh -m generate
```
ซึ่งคำสั่งนี้จะทำการรับไฟล์ ```crypto-config.yaml``` และ ```configtx.yaml``` เข้าไปในคำสั่ง cryptogen และ configtxgen ตามลำดับ

- ```crypto-config.yaml``` file จะแสดง Spec ของ Orderer node และ Peer ต่าง ๆ ใน Organization
- ```configtx.yaml``` จะเป็น spec ในการเชื่อมต่อกันระหว่าง Organization และ Orderer

## ส่วนประกอบของ Hyperledger Network เบื้องต้น
ในระบบของ Hyperledger Fabric จะประกอบไปด้วย Server หลัก ๆ 3 ส่วนคือ

### 1.Application ซึ่งเป็นเหมือน Web Server ที่ยิงคำสั่งเข้าไปใน Endosing Peer เพื่อรับหรือแก้ไขข้อมูลต่าง ๆ
### 2.Peer ซึ่งเป็นตัวแทนขององค์กรหรือ Organization ซึ่งจะรับคำสั่งจาก Application โดยจะแบ่งเป็น 2 ประเภทคือ
#### 2.1 Endosing Peer ทำหน้าที่เช็ค Signature และจำลองการทำงานของคำสั่งจาก Application ออกมาเป็น RW (ReadWrite) Set ส่งกลับไปให้ Application
#### 2.2 Committing Peer ทำหน้าที่ Verify RW Set ว่าตรงกับ World State ปัจจุบันของ Blockchain หรือไม่ 
- ถ้าใช่จะ Commit Block ใหม่​ โดยทำงานตาม RW Set นั้นและแก้ไข World State 
- แต่ถ้าไม่ก็จะ Commit Block ใหม่โดยรวมเอา RW Set นั้นเข้าไปด้วยแต่จะ Mark ว่า Invalid และไม่แก้ไข World State
Orderer เป็นตัวจัดการข้อตกลงร่วมหรือ Consensus ทั้งหมดของระบบซึ่งเป็นตัวหลักแยกออกมาต่างหาก ข้อดีของมันคือทำให้สามารถแก้ไขเปลี่ยนแปลง Consensus ภายหลังได้ง่าย Orderer จะรับคำสั่งของ Application ที่ถูก Endosed จาก Endosing Peer แล้ว ส่งไปให้ Committing Peer ทำการ Verify
โดย Consensus ที่มีในปัจจุบันคือ
- Solo ซึ่งใช้ในระหว่าง Development
- Kafka ใช้ในระบบจริง
- Simplified Byzantine Fault Tolerance (SBFT) กำลังพัฒนาแต่ยังไม่เสร็จ
- Raft มีแผนที่จะพัฒนาเร็ว ๆ นี้

*ปล. Endosing Peer สามารถ Commit Block ได้ แต่ Committing Peer ไม่สามารถ Endorse ได้ และ Orderer สามารถมี Smart Contract ได้แต่ไม่จำเป็น Workflow ของการทำงานของ Fabric

### เมื่อรันคำสั่ง generate แล้วสิ่งที่เราได้มาคือ Private Key และ Certification ของ Organization 1 และ Organization 2 และสร้างไฟล์ Channel Artifacts ดังนี้

- Org1MSPanchors.tx
- Org2MSPanchors.tx
- channel.tx
- genesis.block

ซึ่งเป็นไฟล์ที่ใช้ในการสร้าง Channel บนเครื่อง Peer ขึ้นมา ส่วน genesis.block เป็น Block แรกของ Blockchain ของเรานั่นเอง โดยไฟล์เหล่านี้จะถูก Mount ขึ้นไปอยู่บน Docker ของทุก Peer ของแต่ละ Organization แล้วไปทำงานจริงบนนั้น

จากนั้นเราจะลองทำการสร้าง Network ขึ้นมาจาก Artifacts ทั้งหมดที่เราสร้างขึ้นครับโดยรัน
```
$ bash ./byfn.sh -m up
```

คำสั่งนี้จะเป็นการเรียกใช้ docker-compose ไปที่ไฟล์ docker-compose-cli.yaml ซึ่งจะเรียก Docker ของ Peer ออกมา 4 Containers สำหรับ 2 Organizations และ Orderer อีก 1 Container

และใน Docker เหล่านั้นก็จะทำการ Mount File ที่จำเป็นขึ้นไปด้วย

ยังมี Docker อีกตัวหนึ่งซึ่งชื่อว่า CLI ซึ่งเป็น Peer หนึ่งของ Organization แต่จะทำการรัน Script ใน scripts/script.sh เพื่อทำการทดสอบ Network ทั้งหมดดังนี้

1. Channel ขึ้นมาบนเครื่อง โดยจะได้ไฟล์ mychannel.block
2. ต่อ Channel นี้ขึ้นไปบน Network ผ่าน Orderer
3. อัพเดต Anchor Peer เพื่อให้รู้จักกับ Peer ทั้งหมดใน Channel และคุยกันได้
4. Install Smart Contract หรือที่เรียกว่า Chaincode ซึ่งเขียนโดยภาษา Go ลงไปบน Channel
5. Instantiate หรือการ Initialize ค่าตั้งต้นของ Chaincode
6. Invoke คำสั่ง Query เพื่อรับข้อมูลจาก Blockchain
7. Invoke คำสั่ง Invoke เพื่อแก้ไขข้อมูลบน Blockchain
8. Invoke คำสั่ง Query เพื่อรับข้อมูลจาก Blockchain อีกครั้งเพื่อทดสอบว่าได้แก้ไขจริง

ใน Chaincode ภาษา Go นั้นจะมี 2 functions หลัก ๆ ที่เราต้องเขียนคือ Instantiate และ Invoke โดย

- Instantiate จะทำหน้าที่เซตค่าตั้งต้นของ Chaincode
- Invoke คือคำสั่งที่เหลือที่เราสามารถอนุญาตให้ Application ทำงานได้เช่น Query หรือ Update เวลาทำการ Query จึงเรียกว่า Invoke Update Function นั่นเอง

เมื่อเรารันไปได้สักพักก็จะขึ้นหน้าจอ END แปลว่า Network ของเราได้ทำงานเสร็จสิ้นโดยสมบูรณ์แล้วครับ