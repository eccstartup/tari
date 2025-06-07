---
layout: lesson
title: Adding Tari to Your Exchange
date: 2024-11-06 12:00
author: stringhandler
subtitle:
class: subpage
---

翻译自[官方链接](https://github.com/eccstartup/tari/blob/development/docs/src/09_adding_tari_to_your_exchange.md)

在本指南中，我们将介绍让 Minotari 节点运行的基本知识，并解释安全运行 Minotari 进行交易所需的相关钱包的设置过程：

* 我们将教你如何建立你自己的Minotari节点
* 我们将介绍如何创建 Minotari 钱包作为资金存储，以及相应的设置供监控交易的只读钱包
* 我们将讨论如何监控区块链上的交易
* 我们将涵盖将资金存入钱包和从钱包中提取资金

本指南假设该节点只用作钱包，不会用作挖矿。

## 节点设置
为了接收 Tari，您需要有 Minotari 节点。虽然可以使用具有暴露给 Internet 的正确 gRPC 方法的公共节点，但建议您运行自己的节点。

运行多个节点作为备份以确保可用性也是值得的。

> 注意：对于所有连接到互联网的服务器，它们必须运行 Tor 客户端或配置其公共 IP 信息。关于这一点的文件可在[这里](https://github.com/tari-project/tari#README)和[这里](https://github.com/tari-project/tari/discussions/6366) 。如果你运行在 Linux 上，Tari 应用程序有内置的 Tor 支持，所以可以忽略这一点。

### 第 1 节：创建 Minotari 节点并运行它
Minotari 节点是接收和监视事务所需的base layer节点。

> 注意：如果您使用的是公共 Minotari 节点，则可以跳过本节。请注意，您将需要有问题的公共节点的`公钥`和`公共地址` ，以便正确地进行此交换指南。

1. 在[这里](https://tari.com/downloads/)下载已编译的二进制文件。如果你更喜欢从源代码编译，你需要按照[这里](https://github.com/tari-project/tari#building-from-source)的说明。

2. 使用[此处](https://github.com/tari-project/tari?tab=readme-ov-file#installing-using-binaries)的说明安装二进制文件。

> 注意：根据您的环境，安装文件的位置可能会发生变化。对于 Mac 和 Linux，您可能会在主目录的 `.tari` 文件夹中找到它。它可能是隐藏的，在这种情况下，您需要更改设置才能查看隐藏的文件。在 Windows 上，它将安装在您在安装过程中指定的位置。要让 Minotari 在特定位置创建文件夹，可以使用 `--b` path 命令。请注意，如果您不使用默认文件夹，则需要使用此命令。

安装后你将找到以下二进制文件。

* minotari_console_wallet
* minotari_merge_mining_proxy
* minotari_miner
* minotari_node
* randomx-benchmark
* randomx-codegen
* randomx-tests

进行交易所需的两个文件是 **minotari_node** 和 **minotari_console_wallet**

3. 启动节点（连续运行）：
```
minotari_node
```

如果一个节点尚未创建，它将通知您节点配置文件不存在。你也会被问到你是否想要进行挖矿。在这种情况下选择 `n`。

4. 接下来，系统将询问您是否希望创建节点标识。选择 y。这对于生成私钥/公钥对并让网络识别节点至关重要。

设置完成后，Minotari 基本节点启动。您将看到一个页面，其中列出了各种可用的命令模式（可通过 Ctrl+C 访问）命令。一些有用的是：

* `watch status`: 从命令模式返回到自动刷新状态
* `version`: 您正在运行的 Minotari Node 的版本
* `whoami`: 提供与节点相关的地址信息

5. Type `whoami` and press enter. You'll see your Public Key, Node ID and Public Address, along with a QR Code. You should copy this data to a file or secure location for future reference.

输入 `whoami` 然后按回车键。您将看到您的公钥、节点 ID、公共地址和 QR 码。您应该将此数据复制到文件或安全位置以供将来参考。

```
18:46 v1.0.0-pre.16 esmeralda State: Listening Tip: 3872 (Tue, 23 Jul 2024 14:27:53 +0000) Mempool: 0tx (0g, +/- 0blks) Connections: 0|0 Banned: 0 Messages (last 60s): 0 Rpc: 0/100 ️🔌
>> whoami
Public Key: 90f67a04edcb36261e6304ca213629d183c44e26bd47e38c253473f44d901733
Node ID: e8ed9a4fd38577b6b01e3b8e9d
Public Addresses: /onion3/f5qbkkfkoxowzvshe5mppzpgiiy76cwumpsacungeldoal6hehcgzfqd:18141
Features: PeerFeatures(MESSAGE_PROPAGATION | DHT_STORE_FORWARD)
```

6. 重新启动节点（Ctrl+C 两次退出，然后再次键入 minotari_node）。

### 第2节：创建钱包
在本节中，我们将创建用于接收资金的钱包地址。这个钱包将作为您的 Tari 币的主要存储库。

> NB：这是这个过程中的关键一步。在安全环境中创建钱包并遵循说明对于保护此钱包并防止恶意行为者转移 Tari 非常重要。仔细阅读说明。如果对流程的任何部分有任何疑问，请联系 Tari 社区寻求澄清和帮助。

Minotari 钱包创建过程依赖于seed word phrase（种子短语）来生成相关的主密钥。这个种子短语也允许钱包的恢复。种子短语是从预定义的单词列表中生成的 24 个单词的短语，这些单词将在过程中显示。

> **建议：强烈建议在与任何其他设备或 Internet 断开连接的受信任计算机上执行此过程。在创建钱包以及注意和保护种子短语时应非常谨慎。**

1. 首先，让我们创建一个文件夹来保存所有钱包数据。
```
mkdir tari_wallet_data
cd tari_wallet_data
```

> 注意事项：在钱包创建部分的多个位置，您将被引导复制或记录种子短语，密钥和其他信息。不要将这些文件存储在上面创建的文件夹中，因为您需要在后面的步骤中永久删除此文件夹。

2. 现在开始运行钱包，确保指定 `--base-path` 字段以保留上述文件夹中的所有数据，以便以后可以删除它。
```
minotari_console_wallet --base-path ~/tari_wallet_data
```

3. 你会看到一个菜单。本指南假设您是第一次设置 Minotari，请选择 `1`
```
Console Wallet

1. Create a new wallet.
2. Recover wallet from seed words or hardware device.
3. Create a read-only wallet using a view key.
>>
```

4. 你会被问到你是否想要挖矿。选择 `n`

```
Node config does not exist.
Would you like to mine (Y/n)?
NOTE: this will enable additional gRPC methods that could be used to monitor and submit blocks from this node.
```

5. 系统会询问您是否希望使用已连接的硬件钱包。在这里按 `N`。

```
Would you like to use a connected hardware wallet? (Supported types: Ledger) (Y/n)
```

6. 接下来，系统将要求您输入密码。您需要保存此密码以备将来使用。现在输入这个密码，然后再确认一次。在这样做的时候要小心。我们建议遵循最佳实践来生成强密码。

> 注意：您键入密码时不会看到命令行中的密码。

7. **下一步至关重要。确保没有信息泄漏，并且种子短语仅对您自己和/或受信任方可见。** 输入密码后，您将看到您的种子单词。仔细记下种子词，把它们写下来，并确保它们是安全的。确保您有适当的，同样的安全备份。只有在输入 `confirm` 并按下 `Enter` 键后，您才能继续进行下一步。

```
=========================
       IMPORTANT!        
=========================
These are your wallet seed words.
They can be used to recover your wallet and funds.
WRITE THEM DOWN OR COPY THEM NOW. THIS IS YOUR ONLY CHANCE TO DO SO.

=========================
<...............seed words will be presented here.............>
=========================

I confirm that I will never see these seed words again.
Type the word "confirm" to continue.
>>
```

8. 此时，Minotari 钱包将在控制台界面中启动。

> 注意：以下部分将介绍钱包的配置。虽然没有必要，但额外的安全预防措施是确认您复制的种子单词实际上可以恢复钱包。

### 第3节：获取主钱包的地址

现在我们已经创建了钱包，我们将需要地址`Tari Address one-sided`来创建第二个钱包，该钱包将用于监控交易。

如果您按照上一节的说明进行操作，您应该已经进入 Minotari 控制台钱包界面。如果没有，请运行 `minotari_console_wallet --base-path ~/tari_wallet_data` 并输入密码以启动界面。

1. 在钱包界面中，按两次向右箭头进入 `Receive` 选项卡。此选项卡将列出与钱包关联的所有地址。

![Alt text](./tariexchangeguide_wallet_addresses.png)

2. 复制提供的所有信息，特别注意 `Tari Address one-sided` 字段。这是用户将资金发送到交换的地址。

3. 按 `f10` 或 `Ctrl+Q` 退出钱包

4. 接下来，我们将导出钱包的view key（我们将在**第 4 节**中使用它）。运行以下命令，并在提示时输入钱包密码。

```
minotari_console_wallet --base-path ~/tari_wallet_data export-view-key-and-spend-key
```

您将看到类似于以下内容的信息：

```
1. ExportViewKeyAndSpendKey(ExportViewKeyAndSpendKeyArgs { output_file: None })

View key: cb6c13f07af23380c7756bbfcd622bc3277ec2cc42abd5ed3d8ddd19fa49060c
Spend key: f29039796b3430c6927f26bf216b6241dd7fad7f30a6640e8ac95f3d0af51a52
Minotari Console Wallet running... (Command mode completed)

Press Enter to continue to the wallet, or type q (or quit) followed by Enter.
```

5. 记下 `view key` 和 `spend key`；将它们复制到容易引用的地方。我们将在以后的步骤中要求它们。

6. 键入 `q`，然后按 `Enter` 退出控制台钱包。

7. 确保您已保存上述数据。永久删除文件夹 `tari_wallet_data`，并考虑销毁或安全擦除机器。（译者注：不一定销毁机器这么严重）

> 注意事项：在删除文件夹中的配置数据和/或销毁/擦除设备之前，务必检查记录的密钥、种子短语和地址。

### 第4节：设置只读钱包以接收存款
在本节中，我们将创建第二个钱包，即只读钱包，它将监视在上一节中保存的地址收到的资金。如果你正在进行一笔交易，这是你可以观察收到的资金的方式。这个钱包将需要能够联网。

> 注意：第二个钱包将无法发送任何资金。虽然这限制了钱包使用，但在配置任何可以访问链并与主钱包有某种关联的系统时，保持安全最佳实践是一个很好的做法。

1. 在连接到互联网的服务器上。运行 minotari_console_wallet 命令创建钱包。

> 注意：默认情况下，所有数据都存储在 `~/.tari` 中。你可以在这里找到所有的日志、配置和数据。如果你想使用一个特定的文件夹，你可以使用 `--base-path` 参数来指向一个现有的文件夹或你之前为此目的创建的文件夹。

2. 你会被问到你是否想要挖矿。选择 `N`

```
Node config does not exist.
Would you like to mine (Y/n)?
NOTE: this will enable additional gRPC methods that could be used to monitor and submit blocks from this node.
```

3. 接下来，您将被询问是否要创建新钱包，恢复它，或使用视图密钥创建只读钱包。我们想创建一个只读钱包 ，所以我们在这里选择 `3`。

```
Console Wallet

1. Create a new wallet.
2. Recover wallet from seed words or hardware device.
3. Create a read-only wallet using a view key.
>>
```

4. 接下来我们将被要求输入密码。您需要保存此密码以备将来使用。现在输入此密码并确认。

> 注意：建议您在此处使用与创建第一个钱包时使用的密码不同的密码。

5. You will need to enter the view and spend keys noted in **Section 4**
您需要输入**第 3 节**中提到的`view key`和`spend key`

```
Enter view key:  (hex)
<...view key here...>

Enter the public spend key:  (hex or base58)
<...public spend key here...>  
```

6. 您现在应该看到熟悉的控制台钱包。我们需要在其附带的配置文件中进一步配置它，所以现在按 `f10` 或 `Ctrl+Q` 关闭它并进入下一节。

### 第5节：配置只读钱包
1. 找到 `~/.tari/mainnet/config/config.toml` 配置文件（或您指定的钱包配置文件夹中相应的文件），并在您最喜欢的文本编辑器中打开它。

2. 找到 `Wallet Configuration Options (WalletConfig)`部分。下面是 `config.toml` 文件中钱包配置部分开头的一个典型示例。

```toml
########################################################################################################################
#                                                                                                                      #
#                      Wallet Configuration Options (WalletConfig)                                                     #
#                                                                                                                      #
########################################################################################################################

[wallet]
# The buffer size constants for the publish/subscribe connector channel, connecting comms messages to the domain layer:
# (min value = 300, default value = 50000).
#buffer_size = 50000is
```

3. 接下来，找到行 `#grpc_enabled = false` 并将其更改为 `grpc_enabled = true`。您还需要取消对 `grpc_address` 的注释。

> 注意：如果您希望更安全的 gRPC，您可以编辑其他设置，如 `grpc_authentication`。让钱包的 gRPC 端口不能从公共互联网访问很重要

```toml
# Set to true to enable grpc. (default = false)
grpc_enabled = true
# The socket to expose for the gRPC base node server (default = "/ip4/127.0.0.1/tcp/18143")
grpc_address = "/ip4/127.0.0.1/tcp/18143"
# gRPC authentication method (default = "none")
#grpc_authentication = { username = "admin", password = "xxxx" }
```

4. 设置钱包的基本节点。将此值设置为您在**第 1 节**中创建或选择的 `minotari_node`。

> Note: The format is `<...public key...>::<...public address...>`, with <...> being replaced with the addresses noted previously. Below is a sample of what these configuration settings look like, using the example data from **Section 1**. You should not use the data below, but insert your own details.
注意：格式为 `<...public key...>::<...public address...>`，其中 `<...>` 被替换为之前提到的地址。下面是这些配置设置的示例，使用**第 1 节**中的示例数据。您**不应该**使用下面的数据，而是插入您自己的详细信息。

```toml
# A custom base node peer that will be used to obtain metadata from, example
# "0eefb45a4de9484eca74846a4f47d2c8d38e76be1fec63b0112bd00d297c0928::/ip4/13.40.98.39/tcp/18189"
# (default = )
custom_base_node = "22d33b525d35d256674c5184c262b70d15275effcf5f6fe6dc0d359a18541d04::/onion3/6x54mmubphz5r3opswpuhseswivvlaxbohuqvwsn4o36zmtudq73dgid:18141"
```

5. 保存文件并重新启动钱包。

```
minotari_console_wallet
```

您现在可以接受存款了。在下一节中，我们将解释如何监听传入交易。

### Section 6: Listening for incoming transactions
How you listen for incoming transactions (and what you do with them) will depend on your process. For our example, we'll use the gRPC server that is hosted in the read-only wallet we just created to listen for incoming deposits. 

Reach out to us if you would like an example in your favourite language. You can find more information about the methods available in [wallet.proto here](https://github.com/tari-project/tari/blob/development/applications/minotari_app_grpc/proto/wallet.proto).

```javascript
const grpc = require('@grpc/grpc-js');
const protoLoader = require('@grpc/proto-loader');

// Load the protobuf
const PROTO_PATH = './proto/wallet.proto';
const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
    keepCase: true,
    longs: String,
    enums: String,
    defaults: true,
    oneofs: true
});
const streamingProto = grpc.loadPackageDefinition(packageDefinition).tari.rpc;

// Create a client
console.log(streamingProto);
const client = new streamingProto.Wallet('localhost:18143', grpc.credentials.createInsecure());

const request = {};

// Call the gRPC method
const call = client.GetCompletedTransactions(request);

// Handle the stream of responses
call.on('data', (response) => {
    console.log('Received data:', response);
    // ..... Do business logic with transaction. E.g. compare the reference in payment_id to a reference provided to the exchange client and allocate
    // to their account
    // ....
});

call.on('end', () => {
    console.log('Stream ended.');
});

call.on('error', (err) => {
    console.error('Stream error:', err);
});

call.on('status', (status) => {
    console.log('Stream status:', status);
});
```

This is a basic implementation; some additional items you may want to consider for a production environment are: 

* Using `grpc.credentials.createSsl()` to secure the connection between the wallet and any application calling it. We'll not discuss the process of creating a server key or certificate here; you can read more about the process [here](https://www.ibm.com/docs/en/api-connect/10.0.x?topic=profile-generating-self-signed-certificate-using-openssl). Below is an example:

```javascript
// Load the protobuf definition
const PROTO_PATH = './proto/wallet.proto';
const packageDefinition = protoLoader.loadSync(PROTO_PATH, {
    keepCase: true,
    longs: String,
    enums: String,
    defaults: true,
    oneofs: true
});
const streamingProto = grpc.loadPackageDefinition(packageDefinition).tari.rpc;

// Read the server's certificate (server.crt)
const serverCert = fs.readFileSync('path/to/server.crt'); // Specify the path to your server's certificate

// Create secure credentials for the client using the server's certificate
const credentials = grpc.credentials.createSsl(serverCert);

// Create the gRPC client with the secure credentials
const client = new streamingProto.Wallet('localhost:18143', credentials);

// Prepare the request object (you can modify this based on the method's requirements)
const request = {};

// Call the gRPC method with secure connection
const call = client.GetCompletedTransactions(request);

// Handle the stream of responses
call.on('data', (response) => {
    console.log('Received data:', response);
    // Process transaction data here. Example:
    // Compare the reference in payment_id to a reference provided to the exchange client and allocate to their account
});

call.on('end', () => {
    console.log('Stream ended.');
});

call.on('error', (err) => {
    console.error('Stream error:', err);
});

call.on('status', (status) => {
    console.log('Stream status:', status);
});
```

## Descriptions of Common Activities
### Section 7: An example for receiving funds
Each exchange will have their own processes, but here is an example of receiving funds from a KYC'ed client. 

1. The client begins the deposit process. For example, clicking on a "Deposit" button.

2. The exchange generates a long unique ID for the deposit. This may be a single reference that is reused for the client, or every deposit may have their own reference.

3. The exchange provides their `Tari Address one-sided` address and the reference to the client. The exchange must also save this reference in their internal database.

> Note: Exchanges should use the one-sided or non-interactive addresses so they can receive deposits even if their infrastructure is offline. Interactive addresses are intended for peer-to-peer transactions.

4. The client uses Tari Aurora or another Tari-enabled wallet and sends a non-interactive transaction to the provided address. They must include the provided reference with this transaction.

> Note: Using the Minotari console wallet, for example, the recommendation would be for the user to place your payment reference in the `Payment ID` field.

5. A process similar to the example in **Section 6**, the exchange periodically runs the script to see if there are any new transactions.

6. For new transactions, compare against the list of expected references in their internal database and if there is a match, call the internal system to allocate funds to the client's account.

### Section 8: Performing withdrawals

In this section we'll perform a withdrawal from the same address we used in **Section 3**. It is also possible to have a number of different wallets and send funds between them. The process is mostly the same, but is out of scope for this document.

> NOTE: The wallet used to spend funds should not be online for more time than is necessary. It is recommended that the machine running this wallet is secured.

Before we spend funds, we must have a wallet set up with the seed words created in Step 7 of **Section 2**.

Once the wallet is set up, continue with the steps below.

1. Run the wallet to update the balance

```
minotari_console_wallet --password <password> -p "wallet.custom_base_node=<...node_pub_key...>::<...node_pub_address...>" --auto-exit sync
```

> Note: The custom base node can also be set as an environment variable `TARI_WALLET__CUSTOM_BASE_NODE`

2. Validate there are sufficient funds in the wallet
```
minotari_console_wallet --password <password> -p "wallet.custom_base_node=<...node_pub_key...>::<...node_pub_address...>" --auto-exit get-balance
```

```
Minotari Console Wallet running... (Command mode started)
==============
Command Runner
==============

1. GetBalance

Available balance: 10000.000000 T
Time locked: 0 µT
Pending incoming balance: 27960.980255 T
Pending outgoing balance: 0 µT

Minotari Console Wallet running... (Command mode completed)
```

3. Next, send funds to the desired address.

```
minotari_console_wallet --password <password> -p "wallet.custom_base_node=<node_pub_key>::<node_pub_address>" --auto-exit send-minotari <amount> <destination>
```

Replace `<amount>` and `<destination>` with the amount to send and Tari address to send funds to. Note: The amount is specified in units of 0.000001 XTM. To specify the amount in Tari, you can append the letter `T`. For example, `send-minotari 10000` would send an amount of `0.01 XTM`. `send-minotari 10000T` would send an amount of `10000 XTM`.

Exchanges should not allow clients to provide interactive Tari Addresses. This can be easily validated by checking the second byte of the address. Specifically, the byte that represents an interactive wallet would be 01 in hexadecimal, or 00000001 in binary.

To break it down:
* The value is 01 (hexadecimal)
* In binary, this is 00000001
* The least significant bit (rightmost bit) is 1, indicating support for interactive transactions