<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**链表奇偶分离**



		按照索引
		ListNode* oddEvenList(ListNode* head) {
		    if(head == NULL)    return head;
		    ListNode *odd = head, *even = head->next, *evenStart = even;
		    while(even && even->next)
			odd->next = even->next;
			odd = odd->next;
			even->next = odd->next;
			even = even->next;
		    odd->next = evenStart;//将奇偶链表相连
		return head;
		链表奇偶分离按照数值
		node *dummy_odd=new node(0);node *cur1=dummy_odd;
		node *dummy_even=new node(0node *cur2=dummy_even;
		node *cur=head;node *pre;
		while(cur!=NULL)
		    pre=cur;
		    if(cur->val%2!=0)
			cur1->next=cur;cur1=cur;
		    else
			cur2->next=cur;cur2=cur;
		    if(cur->next==NULL)//避免环路
			cur1->next=NULL;cur2->next=NULL;
		    cur=cur->next;
		pre->next=dummy_even->next;
-	**合并K个排序链表**



		ListNode* mergeKLists(vector<ListNode*>& lists)
		    return mergeKLists(lists,0,lists.size()-1);
		ListNode *mergeKLists(vector<ListNode*>& lists,int left,int right){
		    if(left==right)
			return lists[left];
		    else if(left>right)
			return NULL;
		    int mid=left+(right-left)/2;
		    return mergeTwoLists(mergeKLists(lists,left,mid),mergeKLists(lists,mid+1,right));
		ListNode *mergeTwoLists(ListNode *left,ListNode *right){
		    if(!left) return right;
		    if(!right) return left;
		    if(left->val<=right->val)
			left->next=mergeTwoLists(left->next,right);return left;
		    else
			right->next=mergeTwoLists(left,right->next);
			return right;

-	**复制带随机指针的链表**



		unordered_map<Node*,Node*> memo;
		Node* copyRandomList(Node* head) 
		    if(head==NULL)
			return NULL;
		    if(memo.count(head)>0)
			return memo[head];
		    Node *node=new Node(head->val);
		    memo[head]=node;
		    node->next=copyRandomList(head->next);
		    node->random=copyRandomList(head->random);
		return node;
-	**环形链表以及入口点**



		bool hasCycle(ListNode *head)
		    ListNode *fast=head;
		    ListNode *slow=head;
		    while(fast&&fast->next)
			slow=slow->next;
			fast=fast->next->next;
			if(fast==slow){
			    return true;
		return false;
-	**排序链表**



		ListNode* sortList(ListNode* head)
		    if(head==NULL||head->next==NULL)
			return head;
		    ListNode *pre=head,*slow=head,*fast=head;
		    while(fast&&fast->next)
			pre=slow;
			slow=slow->next;
			fast=fast->next->next;
		    pre->next=NULL;
		    return mergeTwoList(sortList(head),sortList(slow));
		ListNode *mergeTwoList(ListNode *h1, ListNode *h2)
		    if(!h1) return h2;
		    if(!h2) return h1;
		    if(h1->val<h2->val)
			h1->next=mergeTwoList(h1->next,h2);
			return h1;
		    else
			h2->next=mergeTwoList(h2->next,h1);
			return h2;
-	**相交链表**


		ListNode *getIntersectionNode(ListNode *headA, ListNode *headB)
		    ListNode *h1=headA;
		    ListNode *h2=headB;
		    while(h1!=h2)
			if(h1==NULL) h1=headB;
			else 1=h1->next;
			if(h2==NULL) h2=headA;
			else h2=h2->next;
		return h1;
-	**回文链表**


	
		ListNode *front;
		bool isPalindrome(ListNode* head) 
		    front=head;return reverseCheck(head);
		bool reverseCheck(ListNode *cur)
		    if(cur==NULL) return true;
		    bool last=reverseCheck(cur->next);
		    if(!last) return false;
		    if(front->val!=cur->val)return false;
		    front=front->next;
		return true;

-	**反转问题**



		(1)	反转前N个元素
		ListNode successor = null; // 后驱节点
		ListNode reverseN(ListNode head, int n) {
		    if (n == 1) { 
			successor = head.next;
			return head;
		    }
		    ListNode last = reverseN(head.next, n - 1);
		    head.next.next = head;
		    head.next = successor;
		return last;
		(2)	反转m到n之间的元素
		递归：if(m==1)return reverseN head->next=reverseBetween(m-1,n-1);
		非递归写法：
		dummy->next=head;
		ListNode *pre=dummy;
		for(int i=0;i<m-1;i++) pre=pre->next;
		ListNode*cur=pre->next;
		for(int i=0;i<n-m;i++)
		    ListNode *temp=pre->next;
		    pre->next=cur->next;
		    cur->next=cur->next->next;
		pre->next->next=temp;
		(3)K-group反转
		ListNode* reverseKGroup(ListNode* head, int k) {
		    dummy->next=head;
		    int len=this->listLength(head);
		    for(int i=0;i<len/k;i++)
			for(int j=1;j<k;j++)
			    ListNode*temp=pre->next;
			    pre->next=head->next;
			    head->next=head->next->next;
			    pre->next->next=temp;
			pre=head;
			head=head->next;
		    return dummy->next;
