<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**LRU实现**



		struct DLinkedNode {
		    int key, value;
		    DLinkedNode* prev;
		    DLinkedNode* next;
		    DLinkedNode(): key(0), value(0), prev(nullptr), next(nullptr) {}
		    DLinkedNode(int _key, int _value): key(_key), value(_value), prev(nullptr), next(nullptr) {}
		};
		class LRUCache {
		private:
		    unordered_map<int, DLinkedNode*> cache;
		    DLinkedNode* head;
		    DLinkedNode* tail;
		    int size;
		    int capacity;
		public:
		    LRUCache(int _capacity): capacity(_capacity), size(0) {
			head = new DLinkedNode();
			tail = new DLinkedNode();
			head->next = tail;
			tail->prev = head;
		    }
		    int get(int key) {
			if (!cache.count(key)) {
			    return -1;
			}
			// 如果 key 存在，先通过哈希表定位，再移到头部
			DLinkedNode* node = cache[key];
			moveToHead(node);
			return node->value;
		    }
		    void put(int key, int value) {
			if (!cache.count(key)) {
			    // 如果 key 不存在，创建一个新的节点
			    DLinkedNode* node = new DLinkedNode(key, value);
			    // 添加进哈希表
			    cache[key] = node;
			    // 添加至双向链表的头部
			    addToHead(node);
			    ++size;
			    if (size > capacity) {
				// 如果超出容量，删除双向链表的尾部节点
				DLinkedNode* removed = removeTail();
				// 删除哈希表中对应的项
				cache.erase(removed->key);
				// 防止内存泄漏
				delete removed;
				--size;
			    }
			}
			else {
			    // 如果 key 存在，先通过哈希表定位，再修改 value，并移到头部
			    DLinkedNode* node = cache[key];
			    node->value = value;
			    moveToHead(node);
			}
		    }

		    void addToHead(DLinkedNode* node) {
			node->prev = head;
			node->next = head->next;
			head->next->prev = node;
			head->next = node;
		    }

		    void removeNode(DLinkedNode* node) {
			node->prev->next = node->next;
			node->next->prev = node->prev;
		    }

		    void moveToHead(DLinkedNode* node) {
			removeNode(node);
			addToHead(node);
		    }

		    DLinkedNode* removeTail() {
			DLinkedNode* node = tail->prev;
			removeNode(node);
			return node;
		    }
		};
- **矩阵打印问题**

		while(left<=right&&top<=down)
		    for(int col=left;col<=right;col++)
			res.push_back(matrix[top][col]);
		    for(int row=top+1;row<=down;row++)
			res.push_back(matrix[row][right]);
		    if(top<down&&left<right)
			for(int col=right-1;col>left;col--)
			    res.push_back(matrix[down][col]);
			for(int row=down;row>top;row--)
			    res.push_back(matrix[row][left]);
		    left++;right--;top++;down--;
-	**跳跃表**

		class SkipList{
		private:
			struct Node{
				int key;
				Node*pre;
				Node*next;
				Node*top;
				Node*down;
				Node(int _key):key(_key),pre(nullptr),next(nullptr),top(nullptr),down(nullptr){}
			};
		//define skipList basic information
			Node*head;
			int size;
			int level;
		//define basic function
			void addNewNode(Node*x,Node*p);
			void delNode(Node*x);
			Node *searchNode(Node*x);
		public:
			SkipList():head(new Node(INT_MIN)),level(1),size(0){
				srand(time(0));
			}
			~SkipList(){
				delete head;
			}
			void insert(int key);
			void search(int key);
			void remove(int key);
		}
		void SkipList::addNewNode(Node*x,Node*p)
			if(!x->next)
				x->next=p;
				p->pre=x;
			else{
				p->next=x->next;
				x->next->pre=p;
				p->pre=x;
				x->next=p;
		void SkipList::insert(int key)
			Node*p=new Node(key);
			Node*p=head;
			while(true)
				if(x->key<=key)
					if(x->next)
						x=x->next;
					else if(x->down)
						x=x->down;
					else
						break;
				else if(x->pre->down)
					x=x->pre->down;
				else
					x=x->pre;
					break;
			addNewNode(x,p);
			while(rand()%2)
				Node*highp=new Node(key);
				while(!x->top&&x->pre)
					x=x->pre;
				if(x->top){
					x=x->top;
					addNewNode(x,highp);
					highp->down=p;
					p->top=highp;
				else{
					Node *top=new Node(INT_MIN);
					x=top;
					top->down=head;
					head->top=top;
					head=top;
					addNewNode(top,highp);
					highp->down=p;
					p->top=highp;
					++level;
				p=highp;
			++size;
		SkipList::Node *SkipList::searchNode(int key)
			Node *x=head;
			while(ture)
				if(x->val==key)
					return x;
				else if(x->val<key)
					if(x->next)
						x=x->next;
					else if(x->down)
						x=x->down;
					else
						return nullptr;
				else if(x->pre->down)
					x=x->pre->down;
				else
					return nullptr;
		void delNode(Node*x)
			if(!x->next)
				if(x->pre==head&&x->down)
					head=head->down;
					head->top=nullptr;
					delete x->pre;
					--level;
				else
					x->pre->next=nullptr;
			else
				x->pre->next=x->next;
				x->next->pre=x->pre;
-	**前缀树**

		class Trie {
		public:
		    /** Initialize your data structure here. */
		    Trie() {

		    }

		    /** Inserts a word into the trie. */
		    void insert(const string& word) {
			Trie *root=this;
			for(char c:word){
			    if(root->next[c-'a']==nullptr){
				root->next[c-'a']=new Trie();
			    }
			    root=root->next[c-'a'];
			}
			root->is_string=true;
		    }

		    /** Returns if the word is in the trie. */
		    bool search(const string& word) {
			Trie *root=this;
			for(char c:word){
			    if(root->next[c-'a']==nullptr){
				return false;
			    }
			    root=root->next[c-'a'];
			}
			return root->is_string;
		    }
		    /** Returns if there is any word in the trie that starts with the given prefix. */
		    bool startsWith(const string& prefix) {
			Trie *root=this;
			for(char c:prefix){
			    if(root->next[c-'a']==nullptr){
				return false;
			    }
			    root=root->next[c-'a'];
			}
			return true;
		    }
		private:
		    bool is_string=false;
		    Trie *next[26]={nullptr};
		};
-	**二叉树操作**

		查找：
		boolean isInBST(TreeNode root, int target) {
		    if (root == null) return false;
		    if (root.val == target)
			return true;
		    if (root.val < target) 
			return isInBST(root.right, target);
		    if (root.val > target)
			return isInBST(root.left, target);
		    // root 该做的事做完了，顺带把框架也完成了，妙
		}
		插入：
		TreeNode insertIntoBST(TreeNode root, int val) {
		    // 找到空位置插入新节点
		    if (root == null) return new TreeNode(val);
		    // if (root.val == val)
		    //     BST 中一般不会插入已存在元素
		    if (root.val < val) 
			root.right = insertIntoBST(root.right, val);
		    if (root.val > val) 
			root.left = insertIntoBST(root.left, val);
		    return root;
		}
		删除：
		TreeNode deleteNode(TreeNode root, int key) {
		    if (root == null) return null;
		    if (root.val == key) {
			// 这两个 if 把情况 1 和 2 都正确处理了
			if (root.left == null) return root.right;
			if (root.right == null) return root.left;
			// 处理情况 3
			TreeNode minNode = getMin(root.right);
			root.val = minNode.val;
			root.right = deleteNode(root.right, minNode.val);
		    } else if (root.val > key) {
			root.left = deleteNode(root.left, key);
		    } else if (root.val < key) {
			root.right = deleteNode(root.right, key);
		    }
	    return root;
		}
		TreeNode getMin(TreeNode node) {
		    while (node.left != null) node = node.left;
		    return node;
		}
