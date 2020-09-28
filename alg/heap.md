<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**数据流中的中位数**



		void addNum(int num) {
		    if(big_heap.empty()||big_heap.top()>num)
			big_heap.push(num);
		    else
			small_heap.push(num);
		    if(big_heap.size()==small_heap.size()+2)
			small_heap.push(big_heap.top());
			big_heap.pop();
		    if(small_heap.size()==big_heap.size()+1)
			big_heap.push(small_heap.top());
			small_heap.pop();
		double findMedian() 
		    return big_heap.size()==small_heap.size()?(big_heap.top()+small_heap.top())/2.0:big_heap.top();
		private:
		    priority_queue<int, vector<int>, less<int>> big_heap;
			priority_queue<int, vector<int>, greater<int>> small_heap;
- **前K个高频元素**

		struct node
		    int key;
		    int val;
		    node(int k,int v):key(k),val(v){}
		    bool operator<( const node& b) const
			return val<b.val;




-	**滑动窗口最大值**



		vector<int> maxSlidingWindow(vector<int>& nums, int k)
		    int window=0;
		    vector<int> res;
		    for(int i=0;i<nums.size();i++)
			if(max_set.empty()){
			    max_set.push_back(i);
			while(!max_set.empty()&&nums[max_set.back()]<nums[i])
			    max_set.pop_back();
			max_set.push_back(i);
			while(max_set.front()+k<=i)
			    max_set.pop_front();
			if(!max_set.empty()&&i>=k-1)
			    res.push_back(nums[max_set.front()]);
		    return res;
		private:
		deque<int> max_set;
