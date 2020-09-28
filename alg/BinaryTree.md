<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**二叉树前序中序非递归序列**


		vector<int> inorderTraversal(TreeNode* root) {
		    if(root==NULL)
			return {};
		    TreeNode *p=root;
		    stack<TreeNode*> s;
		    while(p!=NULL||!s.empty())
			while(p!=NULL)
			    s.push(p);p=p->left;
			if(!s.empty())
			    p=s.top();
			    res.push_back(p->val);
			    s.pop();p=p->right;
		return res;
-	**二叉树后续遍历序列**



		void postOrderTraversalNew(TreeNode *root,path)
			stack< pair<TreeNode *, bool> > s;
			s.push(make_pair(root, false));
			    bool visited;
			    while(!s.empty())
				root = s.top().first;
				visited = s.top().second;s.pop();
				if(root == NULL)
				    continue;
				if(visited)
				    path.push_back(root->val);
				else
				    s.push(make_pair(root, true));
				    s.push(make_pair(root->right, false));
				    s.push(make_pair(root->left, false));
- **验证二叉搜索树**



		bool isValidBST(TreeNode* root)
		    if(root==NULL)
			return true;
		    return isValidBST(root,NULL,NULL);
		bool isValidBST(TreeNode *root,TreeNode*min,TreeNode*max)
		    if(root==NULL)
			return true;
		    if(min!=NULL&&root->val<=min->val)
			return false;
		    if(max!=NULL&&root->val>=max->val)
			return false;
		return isValidBST(root->left,min,root)&&isValidBST(root->right,root,max);
-	**二叉搜索树是否是对称的**



		bool isSymmetric(TreeNode* root) {
		    if(root==NULL)
			return  true;
		    return isSymmetric(root->left,root->right);
		bool isSymmetric(TreeNode*&left,TreeNode*&right) const{
		    if(left==NULL&&right==NULL)
			return true;
		    if(left==NULL||right==NULL)
			return false;
		return left->val==right->val&&isSymmetric(left->left,right->right)&&isSymmetric(left->right,right->left);
-	**二叉树层序遍历**



		vector<vector<int>> levelOrder(TreeNode* root)
		    if(root==NULL)
			return {};
		    queue<TreeNode*> q; q.push(root); int level=0;
		    while(!q.empty())
			int size=q.size();
			res.push_back({});
			for(int i=0;i<size;i++)
			    TreeNode*temp=q.front(); q.pop();
			    res[level].push_back(temp->val);
			    if(temp->left)
				q.push(temp->left);
			    if(temp->right)
				q.push(temp->right);
			level++;
-	**二叉树最大深度**



		int maxDepth(TreeNode* root)
		    if(root==NULL)
			return 0;
		    int left=maxDepth(root->left);
		    int right=maxDepth(root->right);
		return max(left,right)+1;
-	**前序中序构造二叉树**



		TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) 
		    return buildTree(preorder,0,preorder.size(),inorder,0,inorder.size());
		TreeNode*buildTree(vector<int>& pre,int pre_start,int pre_end, vector<int>& in_,int in_start,int in_end)
		    if(pre_start>=pre_end||in_start>=in_end)
			return NULL;
		    TreeNode *root=new TreeNode(pre[pre_start]);
		    for(int i=in_start;i<in_end;i++)
			if(in_[i]==pre[pre_start])
			    root->left=buildTree(pre,pre_start+1,pre_start+i-in_start+1,in_,in_start,i);
			    root->right=buildTree(pre,pre_start+i-in_start+1,pre_end,in_,i+1,in_end);
		return root;
-	**二叉树最大路径和**



		int maxPathSum(TreeNode* root)
		    if(root==NULL)
			return 0;
		    compute(root);
		    return res;
		int compute(TreeNode *root)
		    if(root==NULL)
			return 0;
		    int left=max(compute(root->left),0);
		    int right=max(compute(root->right),0);
		    res=max(res,root->val+left+right);
		return max(left,right)+root->val;
-	**完全二叉树结点个数**



		int countNodes(TreeNode* root) 
		    if(root==NULL)
			return 0;
		    int l=0,r=0;
		    TreeNode*left=root,*right=root;
		    while(left)//求左右树高度
			left=left->left; l++;
		    if(l==r)
			return pow(2,l)-1;
		    else
			return 1+countNodes(root->left)+countNodes(root->right);

-	**二叉树第K小的结点**



		int kthSmallest(TreeNode* root, int k) {
		    if(root==nullptr)
			return 0;
		    int val1=kthSmallest(root->left,k);
		    if(index==k)
			return val1;
		    index++;
		    if(index==k)
			return root->val;
		    int val2=kthSmallest(root->right,k);
		    if(index==k)
			return val2;
		return 0;
-	**最近公共祖先**



		TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
		    if(root==NULL)
			return root;
		    dfs(root,p,q);
		    return ans;
		bool dfs(tn*root, tn*p,tn*q)
		    if(root==NULL)
			return false;
		    bool left=dfs(root->left,p,q);
		    bool right=dfs(root->right,p,q);
		    if((left&&right)||((left||right)&&(p->val==root->val||q->val==root->val)))
			ans=root;
			return true;
		return left||right||root->val==p->val||root->val==q->val;
-	**序列化与反序列化**



		string rserialize(TreeNode* root){
			if(root==NULL)
			    return "#";
		str=to_string(root->val)+","+rserialize(root->left)+","+rserialize(root->right);
			return str;
		string serialize(TreeNode* root) 
		    string str=rserialize(root);return str;
		TreeNode* rdeserialize(string &data,int& start) {
		    if(data.empty()||data[start]=='#')
			start+=2;
			return NULL;
		    int num=0;int i;int sign=1;
		    if(data[start]=='-')
			sign=-1;start++;
		    for(i=start;i<data.size();i++)
			if(isdigit(data[i]))
			    num=num*10+(data[i]-'0');
			else
			    break;
		    start=i+1;
		    TreeNode *root=new TreeNode(sign*num);
		    if(start<data.size())
			root->left=rdeserialize(data,start);
			root->right=rdeserialize(data,start);
		    return root;
		TreeNode* deserialize(string data)
		    int start=0;
		return rdeserialize(data,start);
-	**路径和为K:不从根节点开始**



		int pathSum(TreeNode* root, int sum)
		    if(root==NULL)
			return 0;
		    dfs(root,sum,0);
		    pathSum(root->left,sum);
		    pathSum(root->right,sum);
		    return res;
		void dfs(TreeNode*root,int sum,int cur)
		    if(root==NULL)
			return;
		    cur+=root->val;
		    if(cur==sum)
			res++;
		    dfs(root->left,sum,cur);
		dfs(root->right,sum,cur);
-	**树中重复的子树**



		vector<TreeNode*> findDuplicateSubtrees(TreeNode* root)
		    dfs(root);
		    return res;
		string dfs(TreeNode*root)
		    if(root==NULL)
			return "";
		    string str=to_string(root->val)+","+dfs(root->left)+","+dfs(root->right);
		    if(memo[str]==1)
			res.push_back(root);
		    memo[str]++;
		return str;
-	**二叉树最大宽度：包括null结点**



		int widthOfBinaryTree(TreeNode* root)
		    if (!root) return 0;
		    queue<pair<TreeNode*, unsigned long long>> q;
		    int ans = 1;
		    q.push({root, 1});
		    while (!q.empty()) 
			int sz = q.size();
			ans = max(int(q.back().second - q.front().second + 1), ans);
			for (int i=0; i < sz; i++) 
			    TreeNode *node = q.front().first;
			    unsigned long long pos = q.front().second;
			    q.pop();
			    if (node->left) q.push({node->left, 2 * pos});
			    if (node->right) q.push({node->right, 2 * pos + 1});
		return ans;
-	**最长同数值路径**



		int longestUnivaluePath(TreeNode* root)
		    helper(root);
		    return ans;
		int helper(TreeNode *root)
		    if(root==NULL)
			return 0;
		    int maxLeft=helper(root->left);
		    int maxRight=helper(root->right);
		if(r->left&root->right&root->val=root->left->val&root->val=root->right->val)
			ans=max(ans,maxLeft+maxRight+2);
		    int maxLength=0;
		    if(root->left&&root->left->val==root->val)
			maxLength=maxLeft+1;
		    if(root->right&&root->right->val==root->val)
			maxLength=max(maxLength,maxRight+1);
		    ans=max(ans,maxLength);
		return maxLength;
-	**判断两棵树是否是镜像**



		bool flipEquiv(TreeNode* root1, TreeNode* root2) {
		    if(root1==NULL&&root2==NULL)
			return true;
		    if(root1==NULL||root2==NULL||root1->val!=root2->val)
			return false;
		    return flipEquiv(root1->left,root2->right)&&flipEquiv(root1->right,root2->left)||
		   flipEquiv(root1->right,root2->right)&&flipEquiv(root1->left,root2->left);
-	**先序构造二叉树**



		TreeNode* bstFromPreorder(vector<int>& preorder)
		    if(preorder.empty())
			return nullptr;
		    return bstFromPreorder(preorder,0,preorder.size()-1);
		TreeNode* bstFromPreorder(vector<int>&preorder,int left, int right)
		    if(left>right)
			return nullptr;
		    TreeNode*root=new TreeNode(preorder[left]);
		    if(left==right)
			return root;
		    int start=left+1;
		    while(start<=right&&preorder[start]<preorder[left]){
			start++;
		    root->left=bstFromPreorder(preorder,left+1,start-1);
		    root->right=bstFromPreorder(preorder,start,right);
		return root;
-	**二叉树结点与祖先之间最大差值**



		int maxAncestorDiff(TreeNode* root)
		    if(root==NULL)
			return 0;
		    dfs(root,root->val,root->val);
		    return res;
		void dfs(TreeNode*root, int min_val, int max_val)
		    if(root==NULL)
			return;
		    res=max(max(abs(root->val-min_val),abs(root->val-max_val)),res);
		    min_val=min(min_val,root->val);
		    max_val=max(max_val,root->val);
		    dfs(root->left,min_val,max_val);
		dfs(root->right,min_val,max_val);
-	**祖父节点为偶数的值**



		int sumEvenGrandparent(TreeNode* root)
		    if(root==NULL)
			return 0;
		    dfs(1,1,root);
		    return res;
		void dfs(int grand_val,int father_val, TreeNode*node)
		    if(node==NULL)
			return;
		    if(grand_val%2==0)
			res+=node->val;
		    dfs(father_val,node->val,node->left);
		dfs(father_val,node->val,node->right);
-	**树中任意两个结点之间的距离**



		int countPairs(TreeNode* root, int distance)
		    dfs(root,distance);
		    return res;
		vector<int> dfs(TreeNode*root, int distance)
		    if(root==NULL)
			return {};
		    if(root->left==NULL&&root->right==NULL)
			return {0};
		    vector<int> ret;
		    vector<int> left=dfs(root->left,distance);
		    for(int &d:left)
			if(++d>distance)
			    continue;
			ret.push_back(d);
		    vector<int> right=dfs(root->right,distance);
		    for(int &d:right)
			if(++d>distance)
			    continue;
			ret.push_back(d);
		    for(int l:left)
			for(int r:right)
			    if(l+r<=distance)
				res+=(l+r<=distance);
		return ret;
-	**二叉树转链表**



		TreeNode* convertBiNode(TreeNode* root)
		    if(root==NULL)
			return NULL;
		    trans(root);
		    while(root->left)
			root=root->left;
		    return root;
		TreeNode* trans(TreeNode*root){
		    if(root==NULL)
			return NULL;
		    TreeNode* left=trans(root->left);
		    if(left!=NULL)
			left->right=root;
		TreeNode* right=trans(root->right);
- **二叉树转链表**



		class Solution
		    Node *head;
		    Node* treeToDoublyList(Node* root)
			if(root==NULL) return NULL;
			Node *pre=NULL;
			treeToDoublyList(root,pre);
			head->left=pre;
			pre->right=head;
			return head;
		    void treeToDoublyList(Node* root,Node*&pre)
			if(root==NULL) return;
			treeToDoublyList(root->left,pre);
			root->left=pre;
			if(pre!=NULL)
			    pre->right=root;
			else
			    head=root;
			pre=root;
			treeToDoublyList(root->right,pre);  
