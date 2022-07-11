### Export Basic variables

`export PATH=~/Desktop/fabric-samples/bin:$PATH` <br />
`export FABRIC_CFG_PATH=${PWD}` <br />
`export CHANNEL_NAME=autochannel` <br />
`export IMAGE_TAG=latest`<br />


### Generate 
cryptogen generate --config=./crypto-config.yaml
configtxgen -profile OrdererGenesis -channelID system-channel -outputBlock ./config/genesis.block
configtxgen -profile AutoChannel -outputCreateChannelTx ./config/$CHANNEL_NAME.tx -channelID $CHANNEL_NAME

docker-compose -f docker-compose.yml up -d
ORDERER_TLS_CA=`docker exec cli  env | grep ORDERER_TLS_CA | cut -d'=' -f2`

docker exec cli peer channel create -o orderer.auto.com:7050 -c $CHANNEL_NAME -f /etc/hyperledger/configtx/$CHANNEL_NAME.tx --tls --cafile $ORDERER_TLS_CA

docker exec cli peer channel join -b $CHANNEL_NAME.block

CORE_PEER_ADDRESS=peer0.manufacturer.auto.com:7051
CORE_PEER_LOCALMSPID=MvdMSP
CORE_PEER_TLS_ENABLED=true
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/server.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/server.key
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/users/Admin@manufacturer.auto.com/msp
ORDERER_TLS_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/auto.com/orderers/orderer.auto.com/msp/tlscacerts/tlsca.auto.com-cert.pem

#### repeat for all the peers###


############### Anchor update ############################################################################################################
configtxgen -profile AutoChannel -outputAnchorPeersUpdate ./config/ManufacturerMSPanchors.tx -channelID autochannel -asOrg ManufacturerMSP

docker cp Org1MSPanchors.tx 91941bd1d86c:/opt/gopath/src/github.com/hyperledger/fabric/peer/
**************Give peer variables inside CLI container************
docker exec cli peer channel update -o orderer.auto.com:7050 -c mychannel -f ./config/Org1MSPanchors.tx --tls --cafile $ORDERER_TLS_CA

################################################Repeat for all orgs###############################################################



##############################Package and install chaincode################################################################################

CORE_PEER_ADDRESS=peer1.manufacturer.auto.com:7051
CORE_PEER_LOCALMSPID=MvdMSP
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer1.manufacturer.auto.com/tls/server.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer1.manufacturer.auto.com/tls/server.key
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer1.manufacturer.auto.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/users/Admin@manufacturer.auto.com/msp
ORDERER_TLS_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/auto.com/orderers/orderer.auto.com/msp/tlscacerts/tlsca.auto.com-cert.pem



peer lifecycle chaincode package fabcar.tar.gz --path /opt/gopath/src/github.com/chaincode/javascript/ --lang node --label fabcar_1

peer lifecycle chaincode install fabcar.tar.gz 

*** copy the package ID ******

fabcar_1:ffd5d937737e1a852402eab332858cac525b79239142b1a9524f982d5d0eec9c


##############################################Repeat this step for all peers and organizations#############################

################################ Approveformyorg########################################
CORE_PEER_ADDRESS=peer0.manufacturer.auto.com:7051
CORE_PEER_LOCALMSPID=MvdMSP
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/server.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/server.key
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/users/Admin@manufacturer.auto.com/msp
ORDERER_TLS_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/auto.com/orderers/orderer.auto.com/msp/tlscacerts/tlsca.auto.com-cert.pem


peer lifecycle chaincode approveformyorg --channelID autochannel --name fabcar --version 1.0 --package-id fabcar_1:ffd5d937737e1a852402eab332858cac525b79239142b1a9524f982d5d0eec9c --sequence 1 --tls --cafile /opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/auto.com/orderers/orderer.auto.com/msp/tlscacerts/tlsca.auto.com-cert.pem --waitForEvent

#############################repeat for all orgs ##########################################################


################################ Commit on all orgs #####################################################

CORE_PEER_ADDRESS=peer0.manufacturer.auto.com:7051
CORE_PEER_LOCALMSPID=ManufacturerMSP
CORE_PEER_TLS_CERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/server.crt
CORE_PEER_TLS_KEY_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/server.key
CORE_PEER_TLS_ROOTCERT_FILE=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/ca.crt
CORE_PEER_MSPCONFIGPATH=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/peerOrganizations/manufacturer.auto.com/users/Admin@manufacturer.auto.com/msp
ORDERER_TLS_CA=/opt/gopath/src/github.com/hyperledger/fabric/peer/crypto/ordererOrganizations/auto.com/orderers/orderer.auto.com/msp/tlscacerts/tlsca.auto.com-cert.pem






peer lifecycle chaincode checkcommitreadiness --channelID autochannel --name fabcar --version 1.0 --sequence 1 --tls --cafile $ORDERER_TLS_CA --output json


peer lifecycle chaincode commit -o orderer.auto.com:7050 --channelID autochannel --name fabcar --tls --cafile $ORDERER_TLS_CA --peerAddresses peer0.manufacturer.auto.com:7051 --tlsRootCertFiles ${PWD}/crypto/peerOrganizations/manufacturer.auto.com/peers/peer0.manufacturer.auto.com/tls/ca.crt --peerAddresses peer0.dealer.auto.com:7051 --tlsRootCertFiles ${PWD}/crypto/peerOrganizations/dealer.auto.com/peers/peer0.dealer.auto.com/tls/ca.crt --peerAddresses peer0.mvd.auto.com:7051 --tlsRootCertFiles ${PWD}/crypto/peerOrganizations/mvd.auto.com/peers/peer0.mvd.auto.com/tls/ca.crt --version 1 --sequence 1 --init-required --signature-policy "AND ('ManufacturerMSP.peer','DealerMSP.peer','MvdMSP.peer')" 



