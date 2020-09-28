<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**归并排序**



		void divide(vector<int> &data,int start,int end)
		    if(start>=end)return;
		    divide(data,start,start+(end-start)/2);
		    divide(data,start+(end-start)/2+1,end);
		    merge(data,start,start+(end-start)/2,end);
		void merge(vector<int> &data,int start,int mid, int end)
		    int i(start),j(mid+1),k(0);
		    int temp[end-start+1];
		    while(i<=mid&&j<=end)
			if(data[i]>data[j])
			    count=(count+mid-i+1)%1000000007;
			    temp[k++]=data[j++];
			else{
			    temp[k++]=data[i++];
		    while(i<=mid) temp[k++]=data[i++];
		    while(j<=end) temp[k++]=data[j++];
		    for(int s=start,i=0;s<end,i<k;s++,i++)
			data[s]=temp[i];
-	**快排**


		Paritition1(int A[], int low, int high) {
		   int pivot = A[low];
		   while (low < high) {
		     while (low < high && A[high] >= pivot) {
		       --high;
		     }
		     A[low] = A[high];
		     while (low < high && A[low] <= pivot) {
		       ++low;
		     }
		     A[high] = A[low];
		   }
		   A[low] = pivot;
		   return low;
		void QuickSort(int A[], int low, int high) //快排母函数
		   if (low < high) 
		     int pivot = Paritition1(A, low, high);
		     QuickSort(A, low, pivot - 1);
		     QuickSort(A, pivot + 1, high);






-	**堆排序**



		void max_heap(vector<int>&v,int root,int heapSize)
		    int left=root*2+1;int right(left+1);int largest=root;
		    if(left<heapSize&&v[left]>v[largest])
			largest=left;
		    if(right<heapSize&&v[right]>v[largest])
			largest=right;
		    if(largest!=root){
			swap(v[root],v[largest]);
			max_heap(v,largest,heapSize);
		void build_max_heap(vector<int> &v,int heapSize)
		    for(int i=heapSize/2;i>=0;i--)
			max_heap(v,i,heapSize);
		void heapSort(vector<int> &v)
		    int heapSize=v.size();
		    build_max_heap(v,heapSize);
		    for(int i=v.size()-1;i>=0;i--)
			swap(v[0],v[i]);
			--heapSize;
			max_heap(v,0,heapSize);
-	**荷兰国旗问题**



		void sortColors(vector<int>& nums)
		    int left=0;
		    int right=nums.size()-1;
		    int cur=0;
		    while(cur<=right)
			if(nums[cur]==0)
			    swap(nums[left++],nums[cur++]);
			else if(nums[cur]==2)
			    swap(nums[right--],nums[cur]);
			else
			    cur++;
-	**煎饼排序**



		vector<int> pancakeSort(vector<int>& A) {
		    int N = A.size();
		    vector<int> res;
		    for (int i = N - 1; i > 0; --i) {
			int j = max_element(A.begin(), A.begin() + i + 1) - A.begin();
			if(j==i){
			    continue;
			}
			if (j > 0) {
			    reverse(A.begin(), A.begin() + j + 1);
			    res.push_back(j + 1);
			}
			reverse(A.begin(), A.begin() + i + 1);
			res.push_back(i + 1);
		    }
		    return res;
		}
-	**冒泡排序**



		int j,k, flag=n;
		while(flag>0)
			k=flag;
			for(j=1;j<k;j++)
				if(a[j-1]>a[j])
					swap(a[j-1],a[j])
					flag=j;

