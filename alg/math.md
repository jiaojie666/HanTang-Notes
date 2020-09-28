<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**大数减法**



		bool cmp ( const vec& a, const vec& b ) 
		    const int na = a.size ( ), nb = b.size ( );
		    if ( na != nb ) return na > nb;   // 两数的数位长度不等
		    for ( int i = na - 1; i >= 0; i-- ) // 两数的数位长度相等
			if ( a [ i ] != b [ i ] )       // 两数对应数位不等
			    return a [ i ] > b [ i ];
		    return true;
		vec sub ( const vec &a, const vec& b ) {
		    const int na = a.size ( ), nb = b.size ( );
		    vec c;
		    int sum = 0;   // -9 <= sum <= 9
		    for ( int i = 0; i < na ; i++ ) {
			sum = a [ i ] - sum;     
			if ( i < nb ) sum -= b [ i ];
			c.push_back ( ( sum + 10 ) % 10 ); 
			( sum < 0 ) ? sum = 1 : sum = 0;   
		    }
		    while ( c.size ( ) > 1 && c.back ( ) == 0 ) c.pop_back();
		    return c;
-	**大数乘法**



		string multiply(string num1, string num2) {
		    int m = num1.size(), n = num2.size();
		    vector<int> res(m + n, 0);
		    for (int i = m - 1; i >= 0; i--)
			for (int j = n - 1; j >= 0; j--) {
			    int mul = (num1[i]-'0') * (num2[j]-'0');
			    int p1 = i + j, p2 = i + j + 1;
			    int sum = mul + res[p2];
			    res[p2] = sum % 10;
			    res[p1] += sum / 10;
			}
		    int i = 0;
		    while (i < res.size() && res[i] == 0)
			i++;
		    string str;
		    for (; i < res.size(); i++)
			str.push_back('0' + res[i]);
		return str.size() == 0 ? "0" : str;





-	**颠倒二进制位**



		uint32_t reverseBits(uint32_t n)
		    int res=0,shift=31;
		    while(n!=0)
			res+=(n&1)<<shift;
			shift--;
			n=n>>1;
		return res;
-	**分数转小数**



		string fractionToDecimal(int numerator, int denominator) {
		    long long int num=static_cast<long long int>(numerator);
		    long long int deno=static_cast<long long int>(denominator);
		    string res;
		    if((num>0)^(deno>0))
			res.push_back('-');
		    num=llabs(num);deno=llabs(deno);
		    res=res+to_string(num/deno);
		    num%=deno;
		    if(num==0)
			return res;
		    res.push_back('.');
		    int index=res.size()-1;
		    unordered_map<int,int> memo;
		    while(num&&memo.count(num)==0)
			memo[num]=++index;
			num*=10;
			res=res+to_string(num/deno);
			num=num%deno;
		    if(memo.count(num)==1)
			res.insert(memo[num],"(");
			res.push_back(')');
		return res;
-	**多数元素**



		int majorityElement(vector<int>& nums) {
		    int count=1;
		    int candidate=nums[0];
		    for(int i=1;i<nums.size();i++)
			if(count==0)
			    candidate=nums[i];
			    count=1;
			else
			    if(nums[i]==candidate)
				count++;
			    else
				count--;
		return candidate;
-	**两数相除**



		int divide(int dividend, int divisor) {
		    long long int sign=1;
		    if(dividend>0^divisor>0)
			sign=-1;
		    long long int divid=abs(dividend);
		    long long int divis=abs(divisor);
		    if(divid<divis)
			return 0;
		    long long int partial_sum=1;
		    long long int cmp=divis;
		    while((cmp<<1)<divid)
			cmp=cmp<<1;
			partial_sum=partial_sum<<1;
		    long long int res=0;
		    while(divid>=divis)
			divid-=cmp;
			res+=partial_sum;
			while(cmp>divid)
			    cmp=cmp>>1;
			    partial_sum=partial_sum>>1;
		return sign*res;
-	**求pow**



		double myPow(double base, long long int exp)
		    if(base==1)
			return base;
		    double ans=1;bool sign=true;
		    if(exp<0)
			sign=false;exp=-exp;
		    while(exp>0)
			if(exp&1)
			    ans*=base;
			exp>>=1;
			base*=base;
		return sign?ans:1/ans;
-	**sqrt**



		float sqrt1(float n) {
			float min, max, mid;
			min = 0;
			max = n;
			mid = n / 2;
			while (mid*mid>n + PRECISION || mid*mid<n - PRECISION)
				mid = (max + min) / 2;
				if (mid*mid < n + PRECISION)
					min = mid; 
				if (mid*mid > n - PRECISION)
					max = mid;
			return mid;
-	**位1的个数**



		int hammingWeight(uint32_t n)
		    int ans=0;
		    while(n)
			n=n&(n-1);
			ans++;
		return ans;
-	**加法**



		int add(int num1, int num2)
		    if(num2 == 0)
			return num1;
		    int sum = num1 ^ num2;
		    int carry = (num1 & num2) << 1;
		    return add(sum, carry);
-	**减法**



		sub=add(~num2,1);
		int result=add(num1,sub);
		return result;
-	**乘法**



		ll multiply(ll a,ll b)
		    if(b<=0)
			return 0;
		    ll left=a;
		    ll right=b;
		    int p=1;
		    while((p<<1)<right)
			p=p<<1;
			left=left<<1;
		return left+multiply(a,b-p);
-	**计算素数**



		isPrimer() for(i=2;i*i<=n;i++) if(!x&1)return false;
		int countPrimes(int n) {
		    vector<bool> isPrim(n,true);
		    for (int i = 2; i * i < n; i++) 
			if (isPrim[i]) 
			    for (int j = i * i; j < n; j += i) 
				isPrim[j] = false;
-	**整数反转**



		int res=0;
		while(x!=0)
		    int pop=x%10;
		    x/=10;
		    if (res > INT_MAX/10 || (res == INT_MAX / 10 && pop > 7)) return 0;
		    if (res < INT_MIN/10 || (res == INT_MIN / 10 && pop < -8)) return 0;
		    res=res*10+pop;
		return res;
-	**数组中只有两个数字只出现一次**



		vector<int> singleNumbers(vector<int>& nums)
		    int ret = 0;
		    for (int n : nums)
			ret ^= n;
		    int div = 1;
		    while ((div & ret) == 0)
			div <<= 1;
		    int a = 0, b = 0;
		    for (int n : nums)
			if (div & n)
			    a ^= n;
			else
			    b ^= n;
		    return vector<int>{a, b};
