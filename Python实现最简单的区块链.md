# Python实现最简单的区块链

## 简介
在逛github的时候看见这个项目[blockchain](https://github.com/dvf/blockchain)，该项目通过python实现了最简单的区块链，我将它复现了一下，并做了一些修改，比如创世区块需要挖矿才生成，而不直接生成，交易时会查看整条区块链确保账号金额足够，简化工作量证明算法等等，同时还加了详细的中文注释，项目地址如下，欢迎给星[myblockchain](https://github.com/ayuLiao/myblockchain)

项目使用的环境：
mac pro，python3.6.3

第三方库：
flask,requests

如果觉得文章太臭太长可以拉到最后的结尾看我瞎扯一下

## 共识
再编写项目前，先有几个共识

+ 1.区块链是通过区块的hash链接起来的，具有不可修改的特性
+ 2.要创建区块必须要进行PoW（工作量证明）
+ 3.区块中存放交易记录、上个区块的hash和PoW
+ 4.区块链网络是分布式、去中心化的，每个网络节点都存有一个区块链
+ 5.每个节点的区块链都一样

如果对上面的共识有疑惑或者不知道说什么，建议去看上一篇文章《比特币原理》，有了这些知识才能更好的理解区块链

## 创建区块链类
创建名为Blockchain类，该类用于存放区块链相关的所有内容，定义如下

```
class Blockchain:
    def __init__(self):
        # 交易列表
        self.current_transactions = []
        # 区块链
        self.chain = []
        # 网络节点
        self.nodes = set()
```

从__init__()方法中可以看到，我们使用current_transactions这个list来存放交易，chain来存放区块链，nodes来存放网络节点

解释一下，首先整个项目通过Flask运行到服务端，客户端通过访问服务器的某些接口进行挖矿、交易数字货币或者检查区块链合法新。这里通过nodes这个set类型变量来存放区块链网络中的节点，set类型确保了其中的数据不重复，我们需要知道区块链网络中的节点，才能进行共识算法的运算，使全网的区块链都一致，而区块链本身我们使用list简单存储一下，实际上区块链本身就是一个数据结构，这里为了简便，才直接使用list的

## 创建新区块
Blockchain类（区块链类）创建好了，就往Blockchain类中加方法吧，首先要编写的new_block()方法，该方法用于创建一个新的区块，并将该区块添加到区块链中，具体定义如下：

```
    def new_block(self, proof, previous_hash):
        '''
        创建一个新区块加入到区块链中
        :param proof: 工作量证明
        :param previous_hash: 上一个区块的hash
        :return: 一个新区块
        '''

        # 区块
        block = {
            # 索引
            'index':len(self.chain) + 1,
            # 时间戳
            'timestamp': time(),
            # 交易账本
            'transactions':self.current_transactions,
            # 工作量证明
            'proof':proof,
            # 上一块区块的hash
            'previous_hash':previous_hash or self.hash(self.chain[-1])
        }
        # 从新置空
        self.current_transactions = []
        # 将区块添加到区块链中，此时交易账本是空，工作量证明是空
        self.chain.append(block)
        return block
```
这个方法的代码很简单，首先创建了block这个dict类型的变量，它存储着一个完整区块所需的所有数据，其中有

+ index:区块唯一索引，标明第几个区块
+ timestamp:创建区块时的时间戳
+ transactions:交易账本，其实记录着要存入这个区块的所有交易信息
+ proof:工作量证明，需要通过穷举法运算出来
+ previous_hash:上一个区块的hash

其中transactions交易账本直接获取self.current_transactions中的值，previous_hash上个区块的hash要么使用参数，要么通过hash方法直接计算出上个区块的hash

这里顺带引出hash()方法的实现，在python是实现hash算法非常简单，导入hashlib这个库则可，hash()方法具体实现如下，该方法也在Blockchain类中，通过staticmethod定义成静态方法，不懂可以看这篇文章[@staticmethod和@classmethod的作用与区别](http://blog.csdn.net/handsomekang/article/details/9615239)

```
@staticmethod
    def hash(block):
        '''
        使用SHA256哈希算法计算区块的哈希
        :param block: 区块
        :return: 区块hash
        '''
        block_string = json.dumps(block, sort_keys=True).encode()
        return hashlib.sha256(block_string).hexdigest()
```
其中使用json的dumps方法将区块这个dict序列化为json格式，注意要使用sort_keys=True，这样可以让json解析后获得的dict通过key排序，还有要使用encode()方法对json字符串进行utf-8编码，不然hashlib.sha256加密时会报编码类型错误

定义完block区块结构后，我们清空了交易list，避免已经加入区块的交易在下次创建新区块时又加入新的区块中。最后将定义好的区块加到chain区块链中

## 创建新交易
在创建新区块时，我们将交易添加到了区块链中，但是到目前为止还没编写创建交易的方法，接着就来编写创建新交易的方法，具体代码如下：

```
        def new_transaction(self, sender, recipient, amount):
        '''
        创建一个新的交易，添加到我创建的下一个区块中
        :param sender: 发送者地址，发送数字币的地址
        :param recipient: 接受者地址，接受数字币的地址
        :param amount: 数字币数量
        :return: 记录本次交易的区块索引
        '''
        money = 0
        if sender != '0':
            for block in self.chain:
                for transactions in block['transactions']:
                    if transactions['sender'] == sender:
                        money -= transactions['amount']
                    if transactions['recipient'] == sender:
                        money += transactions['amount']

            if money < amount:
                return -1

        # 交易账本，可包含多个交易
        self.current_transactions.append({
            'sender':sender,
            'recipient':recipient,
            'amount':amount
        })
        if not self.chain:
            return 0
        # 交易账本添加到最新的区块中
        return self.last_block['index'] + 1
```

创建交易的代码就也很简洁了，无非就是构建一个dict字典，将交易信息存到dict中构建出一个交易，然后将这个交易dict添加到self.current_transactions这个list中，并且返回这个交易将要被存放到的区块索引。

从代码中可以看出，构成一次交易的信息非常简单，就是发送者的地址、接受者的地址、交易时数字货币的数量，但是完成一次交易是有条件的，就是发送者拥有的数字货币数量大于要发送的数量，不然就无法完成交易，因为不够钱，判断也很简单，就是循环整个区块链，将与发送者有关的交易都统计一遍，获得当下发送者拥有的数字货币。如果发送者地址为0，说明是系统奖励给区块建立者的数字货币，就不必做这样的判断。

最后返回要存储这次交易记录的区块索引

## 工作量证明(PoW)算法实现
要创建一个合法的区块就需要进行PoW，PoW大致原理在《比特币原理》那篇文章中也提过，所以就直接来实现一下

```
        def proof_of_work(self, last_block):
        '''
        简单的工作量证明算法
        找到一个数，使得区块的hash前4位为0
        :param last_proof 上一个区块的工作量证明
        :return: 特殊的数
        '''

        # 工作量证明--->穷举法计算出特殊的数
        last_proof = last_block['proof']
        last_hash = self.hash(last_block)
        proof = 0
        while self.valid_proof(proof,last_hash) is False:
            proof += 1

        return proof

    @staticmethod
    def valid_proof(proof, last_hash):
        '''
        验证工作量证明，计算出的hash是否正确
        对上一个区块的proof和hash与当期区块的proof最sha256运算
        :param last_proof: 上一个区块的工作量
        :param proof: 当前区块的工作量
        :param last_hash: 上一个区块的hash
        :return: True 工作量是正确的 False 错误
        '''
        # f-string pyton3.6新的格式化字符串函数
        guess = f'{proof}{last_hash}'.encode()
        guess_hash = hashlib.sha256(guess).hexdigest()
        return guess_hash[:4] == "0000"
```

可以看到代码比较少，注释占多数:-D

工作量证明算法我们定义了两个方法，先看proof_of_work()方法，该方法简单讲就是通过穷举法计算出一个特殊的数，这个特殊的数就是本区块的工作量证明，其中通过valid_proof()方法来判断这个数是否特殊

再看到valid_proof()方法，首先用f函数格式化字符串，f函数是python3.6新添加的函数，它比我们常用的format()格式化函数快一倍，而且更加简单直观，大括号内括着的内容会被替换成相应变量的具体值，然后同样通过hashlib.sha256()来计算hash，然后判断该hash是否前4位为0，如果是，那么就找到了本区块的工作量证明了

在valid_proof()方法中计算工作量证明时只使用了上一个区块的hash，这其实就够了，需要注意的是，工作量证明虽然叫工作量证明，但它不单只是用于证明工作量，我们还需要用它来保护区块链，让区块链就有不可修改的特性，而计算时使用上一个区块的hash就能满足这两个要求，当区块中的数据改变时，该区块的hash也就变了，那么下一个区块就需要重新做工作量证明，这就需要花费大量的算了，因为我们这里只要求hash前4个为0，所以计算相对简单，但现实中没那么轻松，然计算在一个复杂度上才能避免他人算力的攻击。如果在计算区块的工作量证明时不使用上一个区块的hash，而是使用上一个区块中的其他数据，如上一个区块的PoW，虽然这样也能计算出本区块的PoW，但是就无法防御他人的算力攻击了，他人可以修改区块中的数据，但是不改这个区块的PoW，这样他就不用重新计算其他区块的PoW了，虽然区块的hash变了，但变了就变了，因为计算一个区块的hash很快就可以完成，又拥有了目前区块链中所有区块的PoW，完全可以造出一个一样长度的区块链，让网络中的节点难辨真假

## 验证区块链
通过上面的努力，我们已经可以创建区块链了，接着就来编写验证区块链的逻辑，之所以要验证是因为在区块链是分布式去中心的，在区块链网络中会存在其他节点，此时我们无法确定我们手中的区块链就是最长的，需要获取网络中其他节点的区块链，为了避免其他节点发送非法的区块链给我们，就需要验证区块链，定义valid_chain()方法，用于验证区块链

```
    def valid_chain(self, chain):
        '''
        检验区块链是否是合法的
        1.检查区块链是否连续
        2.检查工作量证明是否正常
        :param chain: 一份区块链
        :return: True 区块链合法，False 区块链非法
        '''
        last_block = chain[0] #创世区块
        current_index = 1

        # 循环检查每个区块
        while current_index < len(chain):
            block = chain[current_index]
            print(f'{last_block}')
            print(f'{block}')
            print('\n-----------\n')
            # 本区块存储的hash是否等于上一个区块通过hash函数计算出的hash，判断区块链是否连续
            if block['previous_hash'] != self.hash(last_block):
                return False
            # 验证工作量证明是否计算正确
            if not self.valid_proof(block['proof'], block['previous_hash']):
                return False
            last_block = block
            current_index += 1
        return True
```

验证区块链主要做两个动作

+ 1.检查区块链是否连续，其实就是判断每个区块中存的上一个区块的hash是否真的就是上一个区块的hash
+ 2.检查区块中的PoW是否正确

valid_chain()方法从创世区块开始判断，代码注释都很清楚，就不细讲了

## 添加区块链网络节点
上面一直在说为了确认自身的区块链是否最长，需要向区块链网络中的其他节点请求他们的区块链，下面就来实现这部分逻辑

首先是添加节点，回忆一下，在一开始的__init__()方法中我们定义了set类型的nodes变量用于存放节点，定义register_node()方法，将其他节点的url地址添加进来则可

```
    def register_node(self, address):
        '''
        添加新的节点进入区块链网络，分布式的目的是去中心化
        :param address: 新节点网络地址，如：http://192.168.5.20:1314
        '''
        # urlparse解析url <scheme>://<netloc>/<path>;<params>?<query>#<fragment>
        parsed_url = urlparse(address)
        # 将新节点的url地址添加到节点中
        self.nodes.add(parsed_url.netloc)
```

## 共识算法
共识算法用于回答：网络中有多个节点，如何区别节点间区块链一致性？这个问题，答案也很简单，网络中所有节点都使用最长的合法区块链则可，合法且最长表示该区块链消耗的运算资源最多

定义resolve_conflicts()方法实现共识算法，该方法做的事情也很简单，GET请求每个节点的接口，叫该节点将它的区块链发送给你，你判断一下发送过来的区块链是合法的，再判断一下发过来的区块链和你现在已有的区块链谁长，谁长用谁，具体代码如下

```
    def resolve_conflicts(self):
        '''
        共识算法，确保区块链网络中每个网络节点存储的区块链都是一致的，通过长区块链替换短区块链实现
        :return: True 替换 False不替换
        '''
        neighbours = self.nodes # 邻居节点
        new_chain = None
        # 我拥有区块链的长度
        max_length = len(self.chain)

        for node in neighbours:
            # 访问节点的一个接口，拿到该接口的区块链长度和区块链本身
            try:
                response = requests.get(f'http://{node}/chain')
                if response.status_code == 200:
                    length = response.json()['length']
                    chain = response.json()['chain']

                    # 判断邻居节点发送过来的区块链长度是否最长且是否合法
                    if length > max_length and self.valid_chain(chain):
                        # 使用邻居节点的区块链
                        max_length = length
                        new_chain = chain
            except:
                # 节点没开机
                pass
        if new_chain:
            self.chain = new_chain
            return True
        return False
```

看到网络请求，可能你也就知道Flask的用途了，响应这些请求，必须节点都是在网络上，那么必须有处理请求的功能，使用一个现成的web框架是最简单的做法

到这里Blockchain类就编写完了，还差一个简单的方法，用于获取区块链中最后一个区块，代码如下：

```
@property
    def last_block(self):
        '''
        :return: 区块链中最后一个区块
        '''
        return self.chain[-1]
```

## 创建区块
前面讲了这么久，其实就是定义一个Blockchain类，它提供了很多核心方法，但我们似乎还是没办法创建区块，因为没有些调用new_block()方法的逻辑啊，现在就来写写。

因为整个项目是搭建在Flask上，所以我们定义一个接口专门用于生成新的区块，也就是俗称的挖矿，接口名为mine，具体代码如下

```
@app.route('/mine', methods=['GET'])
def mine():
    '''
    建立新区块
    :return:
    '''
    if not blockchain.chain:
        blockchain.new_transaction(sender='0', recipient=node_identifier, amount=50)
        block = blockchain.new_block(previous_hash='1', proof=100)
        response = {
            'message': '创世区块建立',
            'index': block['index'],
            'transactions': block['transactions'],
            'proof': block['proof'],
            'previous_hash': block['previous_hash'],
        }
        return jsonify(response), 200

    last_block = blockchain.last_block
    proof = blockchain.proof_of_work(last_block)
    # 挖矿获得一个数字货币奖励，将奖励的交易记录添加到账本中，其他的交易记录通过new_transaction接口添加
    blockchain.new_transaction(sender='0', recipient=node_identifier, amount=6)

    previous_hash = blockchain.hash(last_block)
    block = blockchain.new_block(proof, previous_hash)

    response = {
        'message':'新区块建立',
        'index':block['index'],
        'transactions':block['transactions'],
        'proof':block['proof'],
        'previous_hash':block['previous_hash'],
    }

    return jsonify(response),200
```

当有人通过GET方法来访问mine接口时，就会调用mine()方法，mine()方法中的逻辑很简单，首先判断区块链是否存在，不存在，说明要创建创世区块，创建区块时系统会奖励给区块创建者数字货币，代码实现就是通过new_transaction()方法添加一个新的交易，发送者的地址为0，表示系统，接受者通过uuid4()方法生成一个ID，这个ID就唯一表示这个接受者，可以理解成他的钱包地址，uuid4()基于伪随机数来生成一串数字，因为是基于伪随机数，所以生成的这串数字可能会重复的。

交易添加好，就调用new_bloc()方法创建创世区块就好了

如果区块链本来就存在了，说明区块链中已经有区块了，那么就通过区块链中最后一个区块的hash来计算本区块的工作量证明(使用proof_of_work()方法)，然后给在新区块中添加系统奖励的交易记录,最后调用new_block()方法创建出区块

## 创建新交易
现在区块时可以创建了，但是区块上记录的交易只有系统奖励，太单调了，所以接着来编写创建新交易的方法，同样定义一个接口，客户端将交易信息发送到这个接口就可以创建出新的交易，具体代码如下：

```
@app.route('/transactions/new', methods=['POST'])
def new_transaction():
    '''
    将新的交易添加到最新的区块中
    :return:
    '''
    values = request.get_json()

    required = ['sender', 'recipient', 'amount']
    if not all(k in values for k in required):
        return '缺失必要字段',400

    index = blockchain.new_transaction(values['sender'], values['recipient'],values['amount'])
    if index == -1:
        return '余额不足',400
    response = {'message':f'交易账本被添加到新的区块中{index}'}
    return jsonify(response),201
```

首先受到客户端的交易请求信息，判断其中是否具有发起者地址sender、接受者地址recipient、交易数字货币量amount，有才是个合法的请求，然后调用new_transaction()方法创建交易就好了

可能有人会迷惑，创建交易似乎和创建区块分离开了？其实没有，当我们调用new_transaction()方法创建交易时，其实就是往self.current_transactions这个list中添加信息，而调用new_block()方法创建新区块时，会将self.current_transactions存到区块中，完成区块和交易的绑定


## 添加网络节点
不用多说了，区块链要去中心就需要多个节点，当区块链网络上有其他节点时，可以将它注册到进该节点，只有注册了，本节点才能感知到其他节点，同样定义一个接口来注册节点，代码如下

```
@app.route('/nodes/register', methods=['POST'])
def register_nodes():
    values = request.get_json()
    nodes = values.get('nodes')
    if nodes is None:
        return 'Error:提供有效节点列表',400

    for node in nodes:
        blockchain.register_node(node)

    response = {
        'message': '新节点加入区块链网络',
        'total_nodes':list(blockchain.nodes),
    }
    return jsonify(response),201
```

没啥难度，就是解析一下客户端发来的注册节点请求，然后调用register_node()方法注册一下就好了

## 区块链一致性
节点注册好后，就需要确保这些节点中区块链是一致的了，同样定义一个接口来做这个事情，其实就是调用resolve_conflicts()方法

```
@app.route('/node/resolve', methods=['GET'])
def consensus():
    replaced = blockchain.resolve_conflicts()
    if replaced:
        response = {
            'message':'本节点区块链被替换',
            'new_chain':blockchain.chain
        }
    else:
        response={
            'message':'本节点区块链是权威的(区块链网络中最长的)',
            'chain':blockchain.chain
        }
    return  jsonify(response),200
```

最后为了方便，我们定义出一个接口用于展示整个区块链

```
@app.route('/chain', methods=['GET'])
def full_chain():
    '''
    获得完整的区块链
    :return: 整份区块链
    '''
    response = {
        'chain':blockchain.chain,
        'length':len(blockchain.chain),
    }
    return jsonify(response),200
```

最后写一段简单的开启服务器的代码就好了

```
if __name__ == '__main__':
    from argparse import ArgumentParser

    parser = ArgumentParser()
    parser.add_argument('-p','--port', default=5000, type=int, help='服务启动时对应的端口')
    args = parser.parse_args()
    port = args.port
    app.run(host='0.0.0.0', port=port)
```

## 具体效果
到这里，代码就全部编写完成了，可以跑一波看看是否符合预期了，因为项目运行到Flask中，所以要弄个客户端来请求上面定义好的接口，直接使用postman来发送请求就好了

通过下面命令将区块链服务开启，并绑定到5000端口

```
python myblockchain.py -p 5000
```

![](http://obfs4iize.bkt.clouddn.com/%E5%90%AF%E5%8A%A8myblockchain.png)

先使用postman请求chain接口，看看此时节点上有无区块链，发现是空的，因为还没有开始创建区块

![](http://obfs4iize.bkt.clouddn.com/emtpyblock.png)


那么就请求mine接口来创建区块吧，第一次请求mine接口，创建出了创世区块，创世区块中记录了唯一一个交易就是系统奖励，奖励了50个数字货币给创世区块的创造者，比特币的创世区块也是奖励50个币，这里向本聪哥致敬

![](http://obfs4iize.bkt.clouddn.com/1block.png)

继续请求mine接口，创造第二个区块

![](http://obfs4iize.bkt.clouddn.com/2block.png)

第二个区块依旧没有交易，只有系统奖励，6个数字货币

其实这里已经有一个节点地址了，就是f675e46d829a44fc85783a87f1b52284，它创建了2个区块，已经有了56个数字货币，那我让他发5个数字货币给ayuliao，接济一下我这个穷人

![](http://obfs4iize.bkt.clouddn.com/1transactions.png)

这个交易是要写入第三个区块中的，但是此时第三个区块还没被创建出来，那么在第三个区块创建出来前，这些交易都会写到这个区块中，比如我再让他发50个币给我

![](http://obfs4iize.bkt.clouddn.com/2transactions.png)

交易完后，调用mine接口，创建第三块区块，它就包含了刚刚的两个交易记录和系统对区块建立者的奖励

![](http://obfs4iize.bkt.clouddn.com/3block.png)

此时调用chain接口，就可以发现该节点已经有3个区块了

![](http://obfs4iize.bkt.clouddn.com/4block.png)

此时你已经转了55个币给我了，你就剩下一个数字货币了，此时你又想转10币给我是会失败的

![](http://obfs4iize.bkt.clouddn.com/nomoneyblock.png)

为了验证前面编写的共识算法，也就是多节点区块链的一致性，这里再开启一个服务，你可以开在其他电脑，或者同一台电脑的不同端口

```
python myblockchain.py -p 5001
```

开始了5001后，请求5001的mine接口，让5001创建区块，让他创建2个区块，也就是请求两次mine接口，此时5001的区块链长为2，而且跟5000节点的完全不同

![](http://obfs4iize.bkt.clouddn.com/5001block1.png)

此时将5000节点注册到5001节点中，调用5001的/nodes/register接口(你可以通过注册多个节点)

![](http://obfs4iize.bkt.clouddn.com/add_nodes.png)

然后我们来访问5001的node/resolve接口，使用其共识算法，让5001这个节点的区块链与区块链网络上的一直，其实就是判断5001自己手上的区块链跟网络上其他节点的区块链相比是不是最长的，其实我们知道目前为止应该是5000节点拥有的区块链最长，所以5001节点上的区块链会被替换成5000节点上的区块链，以求网络上所有节点的区块链保持一致

![](http://obfs4iize.bkt.clouddn.com/replaceblockchain.png)


到这里这个项目也就表演完了

## 结尾
这个项目虽然简单，但是区块链核心的东西都体现出来了，可以说麻雀虽小，五脏俱全，当前这个项目还是有些失真的，现实中的区块链项目会比这个复杂很多，也正是因为从头开发一个区块链项目是很复杂，才会出现Ethereum(以太坊)这样的平台型区块链项目，它已经将区块链的底层都帮我们实现好了，你可以直接将你的代码通过智能合约的形式写到Ethereum中，这样你的项目凭借Ethereum平台就自动拥有了区块链的所有特性，在Ethereum中虽然智能合约、智能合约的叫，但当你接触后，你就会发现，智能合约并不智能。

还有一点需要提一下，在这个项目中，建立创世区块时，建立者获得了系统50个数字货币的奖励，这50个是可以花费的，但是在现实的比特币中，创世区块奖励的比特币是不能使用的，原因如下

>创世块的收益花不掉，原因如下：比特币客户端把区块和交易分开存贮在两个数据库中，当客户端发现区块数据库为空时，用代码直接生成一个创世块，但是没有生成这个交易，所以客户端中的交易数据库中是没有发送到上述地址这个交易的，因而一旦收到要花掉该收益的交易时，都会拒绝，所以无法得到任何确认，就花不掉这50个币。出现这种情况很可能是中本聪故意的

在创建创世区块时，系统将50个币发送到了这个地址1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa，据说是中本聪本人，虽然这个地址上的比特币不能使用，但是还有有人将比特币源源不断的转到这个账号，事实上，这个创世区块里的地址目前有66.76869249个比特币，是通过1174笔交易发送的，他们将一笔笔钱发送到这个不可交易的账号上可能有点荒谬，但这其实人们（对中本聪）表达感谢的方式

![](http://obfs4iize.bkt.clouddn.com/%E5%88%9B%E4%B8%96block.png)

最后还有点很有意思

>中本聪挖出创世区块花了六天的时间。在早期的比特币论坛上有人猜测，这可能中本聪故意而为之，模仿圣经中关于创世的描述。《创世纪》第2章第2节上记载：“到第七日，上帝结束了他的工作，便休息了。”

这操作，太骚了，我们这等凡人与聪哥差距太大:-D






