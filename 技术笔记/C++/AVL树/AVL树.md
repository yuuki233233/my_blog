>**前言：**
>在掌握二叉搜索树的基础上，我们发现其存在显著缺陷：若插入数据有序，二叉搜索树会退化为单链表，此时增删查改的时间复杂度从`O(logN)`骤降为`O(N)`。为解决这一问题，`G.M.Adelson-Velsky` 和 `E.M.Landis` 在 1962 年提出了**AVL 树**—— 一种自平衡的二叉搜索树。
>
>AVL 树的核心是在二叉搜索树的基础上引入`平衡因子`和`旋转`机制，强制让树的任意节点左右子树高度差不超过 1，从而将树的高度严格控制在`O(logN)`，保证所有操作的时间复杂度稳定在 `O(logN)`。
# 一、AVL树的概念
## 1.1 平衡因子
AVL 树定义**平衡因子（BF）= 右子树高度 - 左子树高度**，且要求**任意节点的平衡因子只能是 -1、0、1**。
- 平衡因子为 0：节点左右子树高度相等（最优状态）；
- 平衡因子为 1：右子树比左子树高 1 层；
- 平衡因子为 - 1：左子树比右子树高 1 层；
- 平衡因子为 ±2：树失衡，必须通过旋转调整
>注：偶数个节点的 AVL 树（如 2、4 节点）无法做到所有节点平衡因子为 0，此时允许高度差为 1，这是 AVL 树的「最优妥协」。

## 1.2 AVL 树的特性
- 继承二叉搜索树的核心规则：`左子树所有节点值 < 根节点值 < 右子树所有节点值`
- 自平衡特性：插入 / 删除后若失衡，通过旋转恢复平衡
- 高度严格可控，接近完美二叉树
>![](图片/AVL树概念01.png)
>![](图片/AVL树概念02.png)
# 二、AVL 树的结构设计
## 2.1 AVL树的结构
对比二叉搜索树，AVL 树的节点需要新增`父节点指针`和`平衡因子`，同时支持键值对存储（更贴近实际应用场景）
**二叉搜索树节点（对比参考）：**
```cpp
// 二叉搜索树节点结构
template<class K>
struct BSTNode
{
	K _key;               // 节点存储的关键码
	BSTNode<K>* _left;    // 左孩子节点指针
	BSTNode<K>* _right;   // 右孩子节点指针

	// 构造函数：初始化节点
	BSTNode(const K& key)
		:_key(key)
		, _left(nullptr)
		, _right(nullptr)
	{
	}
};
```

**AVL 树节点：**
```cpp
template<class K, class V>
struct AVLTreeNode
{
	pair<K, V> _kv;              // 键值对（实用化改造）
	AVLTreeNode<K, V>* _left;    // 左孩子
	AVLTreeNode<K, V>* _right;   // 右孩子
	AVLTreeNode<K, V>* _parent;  // 父节点（旋转时需回溯）
	int _bf;                     // 平衡因子（初始为0）
	
	// 构造函数简化（直接初始化键值对）
	AVLTreeNode(const pair<K, V>& kv)
		:_kv(kv)
		,_left(nullptr)
		,_right(nullptr)
		,_parent(nullptr)
		,_bf(0)
	{ }
};
```
# 三、AVL 树的核心实现
## 3.1 插入操作（核心）
AVL 树插入分为`二叉搜索树规则插入 -> 更新平衡因子 -> 失衡则旋转`三步，是整个AVL树的核心逻辑。
### 3.1.1 插入流程梳理
1. **基础插入**：按二叉搜索树规则找到插入位置，创建新节点并链接父节点；
2. **平衡因子更新**：从插入节点的父节点开始向上回溯，更新每个祖先节点的平衡因子；
3. **平衡检查**：
    - 若平衡因子为 0：子树高度未变化，停止回溯；
    - 若平衡因子为 ±1：子树高度增加，继续向上回溯；
    - 若平衡因子为 ±2：树失衡，执行旋转调整，停止回溯。
### 3.1.2 更新平衡因子
**更新原则：**
- 平衡因子 = 右子树高度 - 左子树高度
- 插入节点，会增加子树高度而影响当前平衡因子，节点插入右子树，parent的平衡因子++，反之亦然
- parent所在子树的高度是否变化决定了是否会继续往上更新
**更新停止条件：**
1. 更新后parent的平衡因子为0，节点的左子树和右子树一样高
2. 更新后parent的平衡因子由`0->1 / 0->-1`，插入前一样高的树，当平衡因子为1时，节点插入右子树，反之亦然
3. 更新后parent平衡因子为`2或-2`，平衡已被破坏，需要旋转降低树的高度
**更新到10节点，平衡因子为2，10所在的子树已不平衡，需要旋转处理**
![](图片/平衡因子01.png)

**更新到中间节点，3为根的子树高度不变，不会影响上一层，更新结束**
![](图片/平衡因子02.png)

**最坏更新到根停止**
![](图片/平衡因子03.png)
### 3.1.3 插入节点及更新平衡因子的代码实现
```cpp
template<class K, class V>
class AVLTree
{
    typedef AVLTreeNode<K, V> Node;
private:
    Node* _root = nullptr; // 根节点初始化

public:
    bool Insert(const K& key, const V& value)
    {
        // 1. 空树直接插入根节点
        if (_root == nullptr)
        {
            _root = new Node(make_pair(key, value));
            return true;
        }

        // 2. 非空树：按BST规则找插入位置
        Node* cur = _root;
        Node* parent = nullptr;
        while (cur)
        {
            if (cur->_kv.first < key) // 键值对比（修正原_kv直接比较的错误）
            {
                parent = cur;
                cur = cur->_right;
            }
            else if (cur->_kv.first > key)
            {
                parent = cur;
                cur = cur->_left;
            }
            else
            {
                // 键已存在，插入失败（符合set/map的特性）
                return false;
            }
        }

        // 3. 创建新节点并链接到父节点
        cur = new Node(make_pair(key, value));
        if (parent->_kv.first < key)
        {
            parent->_right = cur;
        }
        else
        {
            parent->_left = cur;
        }
        cur->_parent = parent; // 维护父节点指针

        // 4. 回溯更新平衡因子 + 平衡检查
        while (parent)
        {
            // 4.1 更新当前父节点的平衡因子
            if (cur == parent->_left)
                parent->_bf--; // 左子树新增节点，BF-1
            else
                parent->_bf++; // 右子树新增节点，BF+1

            // 4.2 平衡因子状态判断
            if (parent->_bf == 0)
            {
                // 平衡因子归0：子树高度未变，无需继续回溯
                break;
            }
            else if (parent->_bf == 1 || parent->_bf == -1)
            {
                // 平衡因子从0→±1：子树高度增加，继续向上更新
                cur = parent;
                parent = parent->_parent;
            }
            else if (parent->_bf == 2 || parent->_bf == -2)
            {
                // 失衡：根据失衡类型旋转
                if (parent->_bf == 2)
                {
                    // 右子树过高：分两种情况
                    if (cur->_bf == 1)
                    {
                        // 右单旋（直线型失衡）
                        RotateL(parent);
                    }
                    else // cur->_bf == -1
                    {
                        // 右左双旋（折线型失衡）
                        RotateRL(parent);
                    }
                }
                else // parent->_bf == -2
                {
                    // 左子树过高：分两种情况
                    if (cur->_bf == -1)
                    {
                        // 右单旋（直线型失衡）
                        RotateR(parent);
                    }
                    else // cur->_bf == 1
                    {
                        // 左右双旋（折线型失衡）
                        RotateLR(parent);
                    }
                }
                // 旋转后平衡恢复，停止回溯
                break;
            }
            else
            {
                // 平衡因子异常（如±3），说明代码逻辑错误
                assert(false && "平衡因子超出合法范围");
            }
        }

        return true;
    }

private:
    // 后续旋转函数放在这里...
};
```
## 3.2 旋转操作
旋转的核心目标：**在保持二叉搜索树规则的前提下，降低失衡子树的高度，恢复平衡**。
旋转分为四种类型：右单旋`(RotateR)`、左单旋`(RotateL)`、左右双旋`(RotateLR)`、右左双旋`(RotateRL)`
### 3.2.1 右单旋（RotateR）
**适用场景**：失衡节点的平衡因子为 - 2，且左孩子的平衡因子为 - 1（左子树的左子树过高，直线型失衡）
**核心逻辑**：将失衡节点的左孩子作为新根，失衡节点作为新根的右孩子，原左孩子的右子树挂到失衡节点的左子树。

**以下5张图解释了右单旋的抽象情况，可以解决大部分右单旋的问题**
![](图片/右单旋图1.png)
![](图片/右单旋图2.png)
![](图片/右单旋图3.png)
![](图片/右单旋图4.png)
![](图片/右单旋图5.png)

```cpp
void RotateR(Node* parent)
{
    Node* subL = parent->_left;    // 失衡节点的左孩子（新根）
    Node* subLR = subL->_right;    // 新根的右子树（需要转移）
    Node* pParent = parent->_parent; // 失衡节点的父节点

    // 1. 转移subLR：挂到parent的左子树
    parent->_left = subLR;
    if (subLR)
        subLR->_parent = parent;

    // 2. 父节点降级：parent作为subL的右孩子
    subL->_right = parent;
    parent->_parent = subL;

    // 3. 链接新根到原父节点
    if (pParent == nullptr)
    {
        // 原parent是根节点，更新根
        _root = subL;
        subL->_parent = nullptr;
    }
    else
    {
        if (pParent->_left == parent)
            pParent->_left = subL;
        else
            pParent->_right = subL;
        subL->_parent = pParent;
    }

    // 4. 重置平衡因子（旋转后子树高度恢复，BF归0）
    parent->_bf = subL->_bf = 0;
}
```
### 3.2.2 左单旋（RotateL）
**适用场景**：失衡节点的平衡因子为 2，且右孩子的平衡因子为 1（右子树的右子树过高，直线型失衡）
```cpp
void RotateL(Node* parent)
{
    Node* subR = parent->_right;    // 失衡节点的右孩子（新根）
    Node* subRL = subR->_left;      // 新根的左子树（需要转移）
    Node* pParent = parent->_parent; // 失衡节点的父节点

    // 1. 转移subRL：挂到parent的右子树
    parent->_right = subRL;
    if (subRL)
        subRL->_parent = parent;

    // 2. 父节点降级：parent作为subR的左孩子
    subR->_left = parent;
    parent->_parent = subR;

    // 3. 链接新根到原父节点
    if (pParent == nullptr)
    {
        _root = subR;
        subR->_parent = nullptr;
    }
    else
    {
        if (pParent->_left == parent)
            pParent->_left = subR;
        else
            pParent->_right = subR;
        subR->_parent = pParent;
    }

    // 4. 重置平衡因子
    parent->_bf = subR->_bf = 0;
}
```
### 3.2.3 左右双旋（RotateLR）
**适用场景**：失衡节点的平衡因子为 - 2，且左孩子的平衡因子为 1（左子树的右子树过高，折线型失衡）。
**核心逻辑**：先对左孩子做左单旋，再对失衡节点做右单旋。

图下右单旋和左右双旋的比较可进行总结：
**右单旋 / 左单旋：** 节点之间成一条直线（右单旋只解决纯粹的左边搞）
**左右双旋 / 右左双旋：** 节点之间成一条曲折的线（但在b子树不单纯是左边高，只是对于10是左边高，对于5是右边高，需旋转两次才能解决。对5为旋转点进行一次左单旋，对10为旋转点进行一次右单旋）
![](图片/左右双旋.png)
 >**对于右单旋：** 旋转结束，各节点平衡因子都为0
 >**对于左右双旋：** 旋转结束，原父节点**左右两子节点平衡因子不一定全为0**，左子节点平衡因子可能为：1、0、-1，右子节点平衡因子可能为：0、1、-1，这时由高度不同会出现3个场景，由下图可知：
 >![](图片/左右双旋结果图01.png)
 >![](图片/左右双旋结果图02.png)
 >**场景1**：`h>=1`时，新增节点插入在e子树，e子树高度从`h-1`并不断更新`8->5->10`平衡因子，引发旋转，其中8的平衡因子为-1，旋转后8和5平衡因子为0，10平衡因子为1
 >**场景2**：`h>=1`时，新增节点插入在f子树，f子树高度从`h-1`变为h并不断更新`8->5->10`平衡因子，引发旋转，其中8的平衡因子为-1，旋转后8和5平衡因子为0，10平衡因子为-1
 >**场景3**：`h==0`时，a/b/c都是空树，b自己就是一个新增节点，不断更新`5->10`平衡因子，引发旋转，其中8的平衡因子为0，旋转后8和10和5平衡因子均为0
```cpp
// 左右双旋
void RotateLR(Node* parent)
{
    Node* subL = parent->_left;
    Node* subLR = subL->_right;
    int bf = subLR->_bf; // 记录旋转中心的BF，用于后续重置

    // 第一步：对subL做左单旋
    RotateL(subL);
    // 第二步：对parent做右单旋
    RotateR(parent);

    // 根据旋转中心的BF，重置所有节点的平衡因子
    if (bf == 0)
    {
        // 旋转中心是新增节点，所有BF归0
        parent->_bf = subL->_bf = subLR->_bf = 0;
    }
    else if (bf == -1)
    {
        // 新增节点在subLR的左子树
        parent->_bf = 1;
        subL->_bf = 0;
        subLR->_bf = 0;
    }
    else if (bf == 1)
    {
        // 新增节点在subLR的右子树
        parent->_bf = 0;
        subL->_bf = -1;
        subLR->_bf = 0;
    }
    else
    {
        assert(false && "旋转中心平衡因子异常");
    }
}
```
### 3.2.4 右左双旋（RotateRL）
**适用场景**：失衡节点的平衡因子为 2，且右孩子的平衡因子为 - 1（右子树的左子树过高，折线型失衡）。
**核心逻辑**：先对右孩子做右单旋，再对失衡节点做左单旋。
```cpp
// 右左双旋
void RotateRL(Node* parent)
{
    Node* subR = parent->_right;
    Node* subRL = subR->_left;
    int bf = subRL->_bf; // 记录旋转中心的BF

    // 第一步：对subR做右单旋
    RotateR(subR);
    // 第二步：对parent做左单旋
    RotateL(parent);

    // 重置平衡因子
    if (bf == 0)
    {
        parent->_bf = subR->_bf = subRL->_bf = 0;
    }
    else if (bf == 1)
    {
        // 新增节点在subRL的右子树
        parent->_bf = -1;
        subR->_bf = 0;
        subRL->_bf = 0;
    }
    else if (bf == -1)
    {
        // 新增节点在subRL的左子树
        parent->_bf = 0;
        subR->_bf = 1;
        subRL->_bf = 0;
    }
    else
    {
        assert(false && "旋转中心平衡因子异常");
    }
}
```
## 3.3 查找操作
继承二叉搜索树的查找逻辑，时间复杂度稳定在 O(logN)
```cpp
// 查找指定键对应的节点（返回值为Node*，方便扩展）
Node* Find(const K& key)
{
    Node* cur = _root;
    while (cur)
    {
        if (cur->_kv.first < key)
        {
            cur = cur->_right;
        }
        else if (cur->_kv.first > key)
        {
            cur = cur->_left;
        }
        else
        {
            // 找到节点，返回指针
            return cur;
        }
    }
    // 未找到，返回空
    return nullptr;
}
```
## 3.4 平衡检测（优化 + 修复）
我们实现的AVL是否合格，我们通过检查左右子树高度差的程序进行反向验证，同时检查一下节点的平衡因子更新是否出现了问题
```cpp
// 辅助函数：计算子树高度
int _Height(Node* root)
{
    if (root == nullptr)
        return 0;
    // 递归计算左右子树高度，取最大值+1
    int leftH = _Height(root->_left);
    int rightH = _Height(root->_right);
    return max(leftH, rightH) + 1;
}

// 核心检测函数：验证是否为合法AVL树
bool _IsBalanceTree(Node* root)
{
    // 空树是合法AVL树
    if (root == nullptr)
        return true;

    // 1. 计算当前节点的实际平衡因子
    int leftH = _Height(root->_left);
    int rightH = _Height(root->_right);
    int actualBF = rightH - leftH;

    // 2. 校验平衡因子合法性
    if (abs(actualBF) >= 2)
    {
        cout << "节点[" << root->_kv.first << "]失衡：实际BF=" << actualBF << endl;
        return false;
    }
    if (root->_bf != actualBF)
    {
        cout << "节点[" << root->_kv.first << "]BF错误：记录=" << root->_bf << "，实际=" << actualBF << endl;
        return false;
    }

    // 3. 递归校验左右子树
    return _IsBalanceTree(root->_left) && _IsBalanceTree(root->_right);
}

// 对外接口：检测整棵树的平衡性
bool IsBalanceTree()
{
    return _IsBalanceTree(_root);
}
```
# 四、AVL 树的总结与拓展
### 4.1 核心优势

1. **极致平衡**：任意节点左右子树高度差≤1，查询效率稳定；
2. **时间复杂度**：增删查改均为 O(logN)，无最坏情况；
3. **原理清晰**：基于二叉搜索树的最小改造，易理解、易实现。

### 4.2 局限性

1. **旋转成本高**：插入 / 删除可能触发多次旋转（最多两次双旋），写操作效率低于红黑树；
2. **空间开销大**：每个节点需存储父指针和平衡因子，空间利用率低；
3. **删除逻辑复杂**：本文未实现删除（需处理更多失衡场景），实际应用中删除操作成本更高。
### 4.3 与红黑树的对比

| 特性   | AVL 树         | 红黑树          |
| ---- | ------------- | ------------ |
| 平衡要求 | 严格平衡（高度差≤1）   | 近似平衡（黑高相等）   |
| 旋转次数 | 插入最多 2 次，删除多  | 插入最多 2 次，删除少 |
| 空间开销 | 高（存 BF + 父指针） | 中（存颜色 + 父指针） |
| 适用场景 | 读多写少          | 读写均衡         |
### 4.4 学习建议

1. 先吃透二叉搜索树的核心规则，再理解 AVL 树的「平衡」本质；
2. 旋转操作一定要画图分析，重点关注「节点链接」和「平衡因子重置」；
3. 实现后通过「平衡检测函数」验证，逐步调试旋转逻辑；
4. 后续可学习红黑树，对比理解「严格平衡」与「近似平衡」的设计思想。