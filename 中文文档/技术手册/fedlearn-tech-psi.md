京东科技联邦学习平台(fedlearn)技术手册
============

ID对齐也称为Private Set Intersection (PSI) ，也叫作Secure Entity Alignment，或者Secure Entity Resolution，或者Privacy-Preserving Entity Alignment等。主要是在保密状态下，对多方输入的ID取交集获得公共ID集。

平台目前支持MD5、RSA、Diffie-Hellman等方式的id对齐

#### 3.2.1 MD5方式的ID对齐
基于 MD5 变换的 支持两方或多方ID对齐，该算法id 对齐分为三个步骤，

>1：发送请求阶段
> 
> master发送对齐请求给各个client，此步骤不涉及数据传输。
> 
>2：加密本地ID阶段
> 
> 各个client收到请求后，将本地的 ID 进行MD5加密后发送给master，因MD5变换的不可逆性质，其他客户端无法从加密后的数据找出原始ID。
>  
>3: 对齐阶段
> 
> master将搜集到的各个client的ID列表取交集，并将结果发送给各个client，后续步骤统一采用。
> 
> **根据上述流程，本地只会传出用户ID， 且用户ID经过了MD5加密，该加密方法不可逆，所以不会存在数据泄露情况。**

算法优点： 该算法效率较高，速度较快，步骤简单，各地对本地ID进行加密后，由master将ID对齐后直接获得公共ID集。

#### 3.2.2 RSA方式的ID对齐

> 基于 RSA 的 id 对齐利用了 RSA 解密运算和取模运算的性质，分为六个步骤，该算法仅支持两方ID对齐。
> 1: master 发送对齐请求，并构造两个全域哈希 H 和 H' 发送给客户端。此步骤不涉及加密数据传输。
> 
> 2: 需要样本对齐的两个客户端，各自使用第一个全域哈希 H 对本地的 id 进行映射，记为 
> H<sub>id<sub>1</sub></sub>, H<sub>id<sub>2</sub></sub> 。
> 其中一个客户端生成 RSA 公钥 (n, e) 和私钥 (n, d) 对，并将公钥发给 master。
> 
> 3: master 将 RSA 公钥转发给第二个客户端。该客户端收到后，为本地每个 id 生成一个随机数 R_id， 用来保护 id。
> 对 R<sub>id</sub> 进行 RSA 加密操作，并将 H<sub>id<sub>2</sub></sub> 与加密结果 (R<sub>id</sub>)<sup>e</sup> mod n 分别对应相乘，
> 得到 y = (H<sub>id</sub> * (R<sub>id</sub>)<sup>e</sup>) mod n 发送给 master。
> 
> 4: master 将加密结果转发给第一个客户端。该客户端对收到的密文进行签名，得到 y' = (y<sup>d</sup>) mod n。
> 同时，对本地的哈希结果 H<sub>id<sub>1</sub></sub> 也进行签名，并再计算一层全域哈希，t<sub>1</sub> = H'((H<sub>id<sub>1</sub></sub>)<sup>d</sup> mod n) 发送给 master。
>
> 5: master 将签名结果 y' 和第一个客户端的t<sub>1</sub> 转发给第二个客户端。该客户端去除 y' 的随机数 R<sub>id</sub>，
> 并再计算一层全域哈希得到 t<sub>2</sub> = H'((H<sub>id<sub>2</sub></sub>)<sup>d</sup> mod n)。
> 此时，第二个客户端就拥有两方的加密双层哈希集合，求交集，即为两方的相同 ID，将筛选出的签名哈希集合发送给　master。
>
> 6: master 将对齐结果发送给第一个客户端，映射得到本地的 ID 集合。
>
> **在上述流程，master 充当了加密数据转发者的角色。客户端不会传出用户 id 原数值，而是传出经过加密/签名和多层哈希映射的结果，所以不会存在数据泄露情况。**
>
>  **上述流程是两方的加密样本对齐过程，在多于两方的情况下，将上述过程扩展至多方安全求交的方法，可实现多方的数据 ID 对齐，但当前尚未实现，计划后续实现。**

算法优点： 在需要对齐的双方ID数量差别很大的情形下，基于RSA的ID对齐算法较有优势，可以一定程度上减小通讯、计算开销。

#### 3.2.3 基于Diffie-Hellman算法的ID对齐

基于 Diffie-Hellman 算法的 ID 对齐建立在一种密钥一致性算法之上，核心是实现类满足连续两次加密操作可以交换顺序的加密算法，通过 Diffie-Hellman 算法构建的密钥可以满足对于用户A和B来说，不论是先使用A的公钥进行加密后使用B的公钥进行加密还是先使用B的公钥进行加密后使用A的工要进行加密，得到的结果是一致的。
Enc<sub>A</sub>(Enc<sub>B</sub>(ID)) = Enc<sub>B</sub>(Enc<sub>A</sub>(ID))；此算法支持两方或者多方的ID对齐请求。

Diffie-Hellman ID 对齐算法一共分为四个步骤
> 
> 1：初始化阶段
> 
> 在初始化阶段，master生成两个全局参数g和n用于后续client端的加密过程，master随机选择一个client作为主动方，将g和n两个全局加密参数传输给主动方，此步骤不涉及加密数据传输。
> 各个client生成自己的随机数，用于后续加密；主动方收到g和n之后对主动方本地的id进行加密生成Enc<sub>A</sub>(ID<sub>A</sub>)，并将加密后的主动方ID发送给master。
> 
> 2：主动方ID的二次加密阶段
> 
> master收到主动方加密后的主动方ID，即 Enc<sub>A</sub>(ID<sub>A</sub>) 后，将 Enc<sub>A</sub>(ID<sub>A</sub>) 以及 g 和 n 发给除主动方外的所有客户端，
> 各个非主动客户端在收到Enc<sub>A</sub>(ID<sub>A</sub>)之后对 Enc<sub>A</sub>(ID<sub>A</sub>) 进行各自非主动客户端的二次加密，生成Enc<sub>P<sub>i</sub></sub>(Enc<sub>A</sub>(ID<sub>A</sub>)))，再对各自非主动客户端的ID进行一次加密, 生成 Enc<sub>P<sub>i</sub></sub>(ID<sub>P<sub>i</sub></sub>) ，并将二者均发送给master。 
> 
> 3：非主动方ID的二次加密以及主动方对齐ID储存阶段
> 
> master收到各个客户端发来的 Enc<sub>P<sub>i</sub></sub>(Enc<sub>A</sub>(ID<sub>A</sub>)) 和 Enc<sub>P<sub>i</sub></sub>(ID<sub>A</sub>) 之后，将其发送至主动方客户端，
> 主动方将各个非主动客户端的 ID 进行二次加密，生成 Enc<sub>A</sub>(Enc<sub>P<sub>i</sub></sub>(ID<sub>P<sub>i</sub></sub>)) 之后；对于每个非主动客户端i，将 Enc<sub>P<sub>i</sub></sub>(Enc<sub>A</sub>(ID<sub>A</sub>))) 和 Enc<sub>A</sub>(Enc<sub>P<sub>i</sub></sub>(ID<sub>P<sub>i</sub></sub>)) 进行匹配，找到主动方和各个客户端i的交集后，将这些交集再次取交集获得最终对齐好的结果，主动方在本阶段在本地客户端储存对齐好的ID， 并将对齐好的二次加密后ID结果以及各个客户端各自二次加密好的全部ID结果返回给master。
> 
> 4：非主动方ID的结果储存阶段
> 
> master将对齐好的二次加密后ID结果以及各个客户端各自二次加密好的全部ID结果发送给各个客户端，
> 各个客户端将解密后的对齐好的 ID 结果储存在各自本地。
> 
算法优点：虽然针对于多方的 ID 对齐，基于 Diffie-Hellman 算法的 ID 对齐依旧可以在分别只对各地 ID 
加密两次的情况下保证安全并获得 ID 对齐结果。

#### 3.2.4 基于Freedman算法的ID对齐

基于 Freedman 协议的 ID 对齐建立在使用加法同态加密对多项式系数进行加密的核心思想上，构建一个以一方 ID 为根的多项式，只有在公共 ID 部分的样本 ID 才能使多项式取值为0；此算法支持两方或多方的 ID 对齐请求。

**目前仅支持数字型ID，允许小数点存在。**

基于 Freedman 协议的 ID 对齐算法一共分为五个步骤

> 1：初始化阶段
> 
> master向各个client发送对齐请求，请求返回各方 ID 长度，
> 各个client对各自需要对齐的 ID 长度加上一个小的随机数用于模糊真实长度。
> 
> 2：主动方求解系数阶段
> 
> master收到各方模糊后的 ID 长度之后选择长度最小的一个作为主动方，发送求解多项式f(x) = ∑ß<sub>u</sub>x<sup>u</sup>请求，
> 主动方收到master请求后使用拉格朗日插值法经过化解后求解得到多项式系数ß<sub>0</sub>, ß<sub>1</sub>, ..., ß<sub>n</sub>，使用Paillier加密算法对各个系数进行加密后，将加密后系数和公钥发送给master
> 
> 3： 非主动方计算阶段
> 
> master收到主动发送过来的加密后系数和公钥之后，将其发送给各个非主动方，
> 各个非主动收到加密系数及公钥后，生成一个随机数 r，并对本地的每一个 ID y<sub>i</sub>计算 r * f(y<sub>i</sub>) + y<sub>i</sub>，在所有本地 ID 的多项式计算结束后，将结果发送回master
> 
> 4： 主动方对齐及 ID 储存阶段
> 
> master收到各个非主动方的发来的多项式计算结果f(y<sub>i</sub>)之后，将其发送给主动方，
> 主动方解密后进行对齐，将对齐好的结果储存在本地客户端后，将其余各方对应的对齐好的索引发回给master。
>
> 5： 非主动方 ID 储存阶段
> 
> master将各方的索引发回给各个非主动方，
> 非主动方根据索引找到对应的原始 ID 之后，将对齐好的 ID 储存在本地客户端。















