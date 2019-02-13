# ethers
ethers的OC库

可以直接导入工程使用，具体方法请看视频教程

ethers提供了5中创建钱包的方式
1、创建随机地址的钱包
Account *account = [Account randomMnemonicAccount];
2、通过私钥创建钱包
Account *account = [Account accountWithPrivateKey:[SecureData hexStringToData:[privateKey hasPrefix:@"0x"]?privateKey:@"privateKey"]];
3、通过助记词创建钱包
Account *account = [Account accountWithMnemonicPhrase:@"mnemonics"];
4、通过keystore创建钱包
[Account decryptSecretStorageJSON:keyStore password:pwd callback:^(Account *account, NSError *NSError) {
}];
