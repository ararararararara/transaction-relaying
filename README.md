# :love_letter: transaction-relaying

## :arrow_forward: 노드가 통신할 때 공유하는 데이터 종류
 1. 블록체인에 포함된 블록 및 트랜잭션
 2. 블록체인에 아직 포함되지 않은 거래(미확인 거래)
 
## :arrow_forward: 미확인 거래
 블록체인에 트랜잭션을 포함하기 위해 블록을 직접 채굴할 필요 없이 
 어떤 주소로 코인을 보내고 싶을 때 트랜잭션을 네트워크에 브로드캐스트 하고 일부 노드를 블록체인으로 마이닝 할 수 있게 한다.
 
## :arrow_forward: 트랜잭션 풀
 
 ### :heavy_exclamation_mark: 미확인거래를 트랜잭션 폴에 저장
 
 ```
 let transactionPool = [];
 ```
 
 ### :heavy_exclamation_mark: post로 거래 전송
 
 ```
 app.post('/sendTransaction', (req, res) => {
        ...
    })
 ```
 ### :heavy_exclamation_mark: 트랜잭션 생성
 
 ```
 const sendTransaction = (address, amount) => {
    const tx = createTransaction(address, amount, getPrivateFromWallet(), getUnspentTxOuts(), getTransactionPool());
    addToTransactionPool(tx, getUnspentTxOuts());
    return tx;
};
 ```
 ## :mega: 브로드캐스팅
  ```
 var MessageType;
(function (MessageType) {
    MessageType[MessageType["QUERY_LATEST"] = 0] = "QUERY_LATEST";
    MessageType[MessageType["QUERY_ALL"] = 1] = "QUERY_ALL";
    MessageType[MessageType["RESPONSE_BLOCKCHAIN"] = 2] = "RESPONSE_BLOCKCHAIN";
    MessageType[MessageType["QUERY_TRANSACTION_POOL"] = 3] = "QUERY_TRANSACTION_POOL";
    MessageType[MessageType["RESPONSE_TRANSACTION_POOL"] = 4] = "RESPONSE_TRANSACTION_POOL";
})(MessageType || (MessageType = {}));
 ```

## :arrow_forward: 수신된 미확인 거래 확인

```
const isValidTxForPool = (tx, aTtransactionPool) => {
    const txPoolIns = getTxPoolIns(aTtransactionPool);
    const containsTxIn = (txIns, compareTxIn) => {
        return _.find(txIns, ((txIn) => {
            return txIn.txOutIndex === compareTxIn.txOutIndex && txIn.txOutId === compareTxIn.txOutId;
        }));
    };
    for (const txIn of tx.txIns) {
        if (containsTxIn(txPoolIns, txIn)) {
            console.log('txIn already found in the txPool');
            return false;
        }
    }
    return true;
};
 ```
 
## :arrow_forward: 트랜잭션 풀에서 블록체인으로
 
 ```
 const generateNextBlock = () => {
    const coinbaseTx = getCoinbaseTransaction(getPublicFromWallet(), getLatestBlock().index + 1);
    const blockData = [coinbaseTx].concat(getTransactionPool());
    return generateRawNextBlock(blockData);
};
 
 ```
 ## :arrow_forward: 트랜잭션 풀 업데이트
  ```
 const updateTransactionPool = (unspentTxOuts) => {
    const invalidTxs = [];
    for (const tx of transactionPool) {
        for (const txIn of tx.txIns) {
            if (!hasTxIn(txIn, unspentTxOuts)) {
                invalidTxs.push(tx);
                break;
            }
        }
    }
    if (invalidTxs.length > 0) {
        console.log('removing the following transactions from txPool: %s', JSON.stringify(invalidTxs));
        transactionPool = _.without(transactionPool, ...invalidTxs);
    }
};
 ```


