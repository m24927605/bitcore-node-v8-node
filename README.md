Ubuntu 16.04 OS

Step1  
//Install node.js,the node .js version is v8.11.3. and  npm version 5.6.0  
```curl -sL https://deb.nodesource.com/setup_8.x | sudo -E bash -```  
```sudo apt-get install -y nodejs```  

Step2  
//Install mongoDB and run it
https://docs.mongodb.com/manual/tutorial/install-mongodb-on-ubuntu/
```sudo service mongod start```

Step3  
//Install bitcoinABC's bitcoind (version is 0.17.2)   
```sudo apt-get install software-properties-common```  
```sudo add-apt-repository ppa:bitcoin-abc/ppa```  
```sudo apt-get update```  
```sudo apt-get install bitcoind```  


Step4 
//run bitcoind   
```cd ~```  
```touch bitcoin.conf```  
```vi bitcoin.conf```  
```  
datadir=[where you want to store the node data]
server=1
whitebind=127.0.0.1:8333
whitelist=127.0.0.1
txindex=1
addressindex=1
timestampindex=1
spentindex=1
zmqpubrawtx=tcp://127.0.0.1:28332
zmqpubrawtxlock=tcp://127.0.0.1:28332
zmqpubhashblock=tcp://127.0.0.1:28332
rpcallowip=127.0.0.1
rpcport=8332
rpcuser=bch
rpcpassword=local321
```
```bitcoind -conf=/home/[user]/bitcoin.conf```  

Step5  
//Install bitcore-node  
```sudo apt-get install libzmq3-dev build-essential```  
```git clone -b v8.0.0 https://github.com/bitpay/bitcore-node.git```  
```cd bitcore-node```  
```npm i ajv --save```  
```npm i hawk --save```  
```npm i --save```  

Step5
//change the code to connect the local p2p node  
```cd ~/bitcore-node/src```  
```vi config.ts```
```
import * as os from "os";
import parseArgv from "./utils/parseArgv";
import ConfigType from "./types/Config";
let program = parseArgv([], ["config"]);

function findConfig(): ConfigType | undefined {
  let foundConfig;
  const envConfigPath = process.env.BITCORE_CONFIG_PATH;
  const argConfigPath = program.config;
  const configFileName = "bitcore.config.json";
  let bitcoreConfigPaths = [
    `${os.homedir()}/${configFileName}`,
    `../../../${configFileName}`,
    `../${configFileName}`
  ];
  const overrideConfig = argConfigPath || envConfigPath;
  if (overrideConfig) {
    bitcoreConfigPaths.unshift(overrideConfig);
  }
  // No config specified. Search home, bitcore and cur directory
  for (let path of bitcoreConfigPaths) {
    if (!foundConfig) {
      try {
        const bitcoreConfig = require(path) as { bitcoreNode: ConfigType };
        foundConfig = bitcoreConfig.bitcoreNode;
      } catch (e) {
        foundConfig = undefined;
      }
    }
  }
  return foundConfig;
}

function setTrustedPeers(config: ConfigType): ConfigType {
  for (let [chain, chainObj] of Object.entries(config)) {
    for (let network of Object.keys(chainObj)) {
      let env = process.env;
      const envString = `TRUSTED_${chain.toUpperCase()}_${network.toUpperCase()}_PEER`;
      if (env[envString]) {
        let peers = config.chains[chain][network].trustedPeers || [];
        peers.push({
          host: env[envString],
          port: env[`${envString}_PORT`]
        });
        config.chains[chain][network].trustedPeers = peers;
      }
    }
  }
  return config;
}
const Config = function(): ConfigType {
  let config: ConfigType = {
    maxPoolSize: 20,
    pruneSpentScripts: true,
    port: 3000,
    dbHost: process.env.DB_HOST || "127.0.0.1",
    dbName: process.env.DB_NAME || "bitcore",
    numWorkers: os.cpus().length,
    chains: {}
  };

  let foundConfig = findConfig();
  Object.assign(config, foundConfig, {});
  if (!Object.keys(config.chains).length) {
    Object.assign(config.chains, {
      BCH: {
        mainnet: {
          chainSource: "p2p",
          trustedPeers: [{ host: "127.0.0.1", port: 8333 }],
          rpc: {
            host: "127.0.0.1",
            port: 8332,
            username: "bch",
            password: "local321"
          }
        }
      }
    });
  }
  config = setTrustedPeers(config);
  return config;
};

export default Config();

```


Step7(options)  
//Install pm2  
```cd ~```  
```sudo npm install pm2 -g```  
//If the server reboot,pm2 will auto restart.  
```pm2 startup```  
```sudo env PATH=$PATH:/usr/bin /usr/lib/node_modules/pm2/bin/pm2 startup systemd -u ubuntu --hp /home/ubuntu```  
```pm2 save```  

//if you install pm2 please choose step7.1 to run the bitcoin-cash server.  
Step 8.1  
```cd ~/bicore-node```   
```pm2 start "/usr/bin/npm" --name "BCHAPISERVER" -- start```  

//if you didin't install pm2,please choose step7.2 to run the bitcoin-cash server.   
Step8.2  
```cd ~/bitcore-node```  
```npm start```

//////Done!!!!///////

