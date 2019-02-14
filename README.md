# ethers
ethers的OC库

可以直接导入工程使用，具体方法请看视频教程

ethers提供了5中创建钱包的方式

// 1、创建随机地址的钱包
Account *account = [Account randomMnemonicAccount];

// 2、通过私钥创建钱包
Account *account = [Account accountWithPrivateKey:[SecureData hexStringToData:[privateKey hasPrefix:@"0x"]?privateKey:@"privateKey"]];

// 3、通过助记词创建钱包
Account *account = [Account accountWithMnemonicPhrase:@"mnemonics"];

// 4、通过keystore创建钱包
[Account decryptSecretStorageJSON:keyStore password:pwd callback:^(Account *account, NSError *NSError) {
}];

NSString *gasPrice = @"100000000000"; // 手动设置传入参数
NSString *decimal = @"18"; // wei
Transaction *transaction = [Transaction transactionWithFromAddress:[Address addressWithString:@""]];

// 5、获取nonce

EtherscanProvider *e = [[EtherscanProvider alloc]initWithChainId:ChainIdHomestead apiKey:@"RCWEX6WYBXMJZHD5FD617NZ99TZADKBEDJ"];
[[e getTransactionCount:transaction.fromAddress] onCompletion:^(IntegerPromise *pro) {
    if (pro.error != nil) {
        NSLog(@"%@获取nonce失败",pro.error);

    }else{

        NSLog(@"3 获取nonce成功 值为%ld",pro.value);
        transaction.nonce = pro.value;
    }
}];

// 6、获取gasPrice

[[e getGasPrice] onCompletion:^(BigNumberPromise *proGasPrice) {
    if (proGasPrice.error == nil) {

        NSLog(@"5 获取gasPrice成功 值为%@",proGasPrice.value.decimalString);
    }else {

        if (gasPrice == nil) {

            transaction.gasPrice = proGasPrice.value;
        }else{
            NSLog(@"手动设置了gasPrice = %@",gasPrice);

            transaction.gasPrice = [[BigNumber bigNumberWithDecimalString:gasPrice] mul:[BigNumber bigNumberWithDecimalString:@"1000000000"]];
        }

        transaction.chainId = e.chainId;
        transaction.toAddress = [Address addressWithString:@""];
        transaction.value = [[Payment parseEther:@"100"] div:[BigNumber bigNumberWithInteger:pow(10.0, 18 - decimal.integerValue)]];
    }
}];

// 7、合约签名

SecureData *data = [SecureData secureDataWithCapacity:68];
[data appendData:[SecureData hexStringToData:@"0xa9059cbb"]];

NSData *dataAddress = transaction.toAddress.data;//转入地址（真实代币转入地址添加到data里面）
for (int i=0; i < 32 - dataAddress.length; i++) {
    [data appendByte:'\0'];
}
[data appendData:dataAddress];

NSData *valueData = transaction.value.data;//真实代币交易数量添加到data里面
for (int i=0; i < 32 - valueData.length; i++) {
    [data appendByte:'\0'];
}
[data appendData:valueData];

transaction.value = [BigNumber constantZero];
transaction.data = data.data;
transaction.toAddress = [Address addressWithString:@"toAdress"];//合约地址
[account sign:transaction];
NSData *signedTransaction = [transaction serialize];

// 8、发起转账

[[e sendTransaction:signedTransaction] onCompletion:^(HashPromise *pro) {

    NSLog(@"CloudKeychainSigner: Sent - signed=%@ hash=%@ error=%@", signedTransaction, pro.value, pro.error);

    if (pro.error == nil){
        NSLog(@"\n---------------【生成转账交易成功！！！！】--------------\n哈希值 = %@\n",transaction.transactionHash.hexString);
        NSLog(@" 7成功 哈希值 =  %@",pro.value.hexString);

        [[e getTransaction:pro.value]onCompletion:^(TransactionInfoPromise *info) {
            if (info.error == nil) {
                NSLog(@"===%@",info.value.transactionHash.hexString);
            }else{

                NSLog(@" 9查询哈希%@失败 %@",pro.value.hexString,pro.error);
            }
        }];

    }else{
        NSLog(@" 8转账失败 %@",pro.error);
    }
}];
