<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

-	**无重复字符最长子串**


		unordered_map<char,int> map;
		int lengthOfLongestSubstring(string s)
		    if(s.empty())
			return 0;
		    int max=1,right=0,left=0;
		    map[s[right]]=1;
		    while(right<s.size()){
			right++;
			if(map[s[right]]<=1)
			    map[s[right]]++;
			if(right-left>max)
			    max=right-left;
			while(map[s[right]]>1)
			    map[s[left++]]--;
		return max;
-	**字符串Z字形状打印**



		string convert(string s, int numRows)
		    if(numRows==1)
			return s;
		    vector<string> vec(numRows);
		    int curRow=0;
		    bool down=false;
		    for(char c:s)
			vec[curRow]+=c;
			if(curRow==0||curRow==numRows-1)
			    down=!down;
			curRow+=(down?1:-1);
		    string res;
		    for(string str:vec)
			res+=str;
		return res;


-	**字符串转整数**



		class stageChange{
		public:
		    int sign=1;
		    long long int ans=0;
		    void get(char c){
			state=table[state][getCol(c)];
			if(state=="number")
			    ans=ans*10+c-'0';
			    ans = sign == 1 ? min(ans, (long long)INT_MAX) : min(ans, -(long long)INT_MIN);
			else if(state=="sign")
			    sign=c=='-'?-1:1;
		private:
		    string state="space";
		    unordered_map<string,vector<string>> table{
			{"space",{"space","sign","number","other"}},
			{"sign",{"other","other","number","other"}},
			{"number",{"other","other","number","other"}},
			{"other",{"other","other","other","other"}}
		    };
		    int getCol(char c)
			if(isspace(c)) return 0;
			else if(c=='+'||c=='-')return 1;
			else if(isdigit(c))return 2;
			else return 3;
		int myAtoi(string str) 
		    stageChange sc;
		    for(char c:str)
			sc.get(c);
		return sc.ans*sc.sign;
-	**罗马数字转整数**



		int romanToInt(string s)
		    int ans=0;
		    for(int i=0;i<s.size();i++)
			if(i+1<s.size()&&map[s[i]]<map[s[i+1]])
			    ans-=map[s[i]];
			else
			    ans+=map[s[i]];
		    return ans;
		unordered_map<char,int> map={
		    {'I',1},{'V',5},{'X',10},{'L',50},
		{'C',100}, {'D',500}, {'M',1000}};
-	**最长公共前缀**



		string longestCommonPrefix(vector<string>& strs)
		    if(strs.empty())
			return "";
		    return longestCommonPrefix(strs,0,strs.size()-1);
		string longestCommonPrefix(vector<string>& strs,int left,int right)
		    if(left==right)
			return strs[left];
		    int mid=left+(right-left)/2;
		    string l=longestCommonPrefix(strs,left,mid);
		    string r=longestCommonPrefix(strs,mid+1,right);
		    return commonStr(l,r);
		string commonStr(const string& lcpLeft,const string& lcpRight) 
		    int minLength = min(lcpLeft.size(), lcpRight.size());
		    for (int i = 0; i < minLength; ++i) 
			if (lcpLeft[i] != lcpRight[i]) 
			    return lcpLeft.substr(0, i);
		return lcpLeft.substr(0, minLength);



-	**电话号码的组合**



		vector<string> letterCombinations(string digits)
		    if(digits.empty())
			return {};
		    string s;
		    backtrack(0,digits.size(),digits,s);
		    return res;
		void backtrack(int cur,int end,string digits,string str){
		    if(cur>=end)
			res.push_back(str);
			return;
		    int key=digits[cur]-'0'-1;
		    string mVal=map[key];
		    for(int i=0;i<mVal.size();i++){
			str+=mVal[i];
			backtrack(cur+1,end,digits,str);
			str.pop_back();
		vector<string> map={"","abc","def","ghi","jkl","mno","pqrs","tuv","wxyz"};
-	**字母异位词**



		unordered_map<string,vector<string>> res;
		vector<vector<string>> groupAnagrams(vector<string>& strs) {
			for(string &str:strs)
			    vector<int> count(26,0);
			    for(char c:str)
				count[c-'a']++;
			    string s;
			    for(int i=0;i<26;i++)
				s+=to_string(count[i]);
			    if(res.count(s)==0)
				res[s]={str};
			    else
				res[s].push_back(str);
-	**最小覆盖子串**



		string minWindow(string s, string t)
		    string res = "";
		    int m[128] = { 0 };
		    int left = 0, right = 0, need = t.size(), minstart = 0, minlen = INT_MAX;
		    for(char c : t)     
			++m[c];
		    while(right < s.size())
			if(m[s[right]] > 0)    --need;
			--m[s[right]];
			++right;
			while(need == 0) 
			    if(right - left < minlen)
				minstart = left;
				minlen = right - left;
			    ++m[s[left]];
			    if(m[s[left]] > 0)  ++need;
			    ++left;
		    if(minlen != INT_MAX)   res = s.substr(minstart , minlen);
		return res;
-	**判断字符串是否是数字**



		string pattern="[+-]?(\\d*\\.\\d+|\\d+\\.?)([Ee][+-]?\\d+)?";
		regex re{pattern};
		string s(str,strlen(str));
		return regex_match(s,re);


-	**strStr**



		int strStr(string haystack, string needle)
		    int n_size=needle.size();
		    int h_size=haystack.size();
		    long long int h_hash=0;
		    long long int n_hash=0;
		    long long int high=1;
		    for(int i=0;i<n_size-1;i++)
			high=(high*base)%mod;
		    for(int i=0;i<n_size;i++)
			h_hash=(h_hash*base+haystack[i])%mod;
			n_hash=(n_hash*base+needle[i])%mod;
		    int start=0;
		    while(start<=h_size-n_size)
			if(n_hash==h_hash&&haystack.substr(start,n_size)==needle)
			    return start;
			else
			    h_hash=(base*(h_hash-(haystack[start])*high)+haystack[start+n_size])%mod;
			start++;
		    return -1;
		long long int mod=1000000001;
		long long int base=256;
		int strStr(string haystack, string needle)
		    int hSize=haystack.size();
		    int nSize=needle.size();
		    unordered_map<char, int> pianyi;
		    for(int i=0;i<nSize;i++) pianyi[needle[i]]=nSize-i;
		    int i=0;
		    while(i<=hSize-nSize)
			if(haystack.substr(i,nSize)==needle) return i;
			else{
			    if(i+nSize>hSize-1) return -1;
			    else
				if(pianyi.find(haystack[i+nSize])!=pianyi.end())
				    i+=pianyi[haystack[i+nSize]];
				else
				    i+=nSize+1;
		return -1;
-	**实现计算器**



		int helper(string &s)
		{
		    stack<int> stk;
		    int num = 0;
		    char sign = '+';
		    while(s.size()>0){
			char c = s[0];
			s.erase(s.begin());
			if (isdigit(c))
			    num = 10 * num + (c - '0');
			if(c=='('){
			    num=helper(s);
			}
			if (!isdigit(c)&&c!=' '|| s.size()==0) {
			    switch (sign) {
				int pre;
				case '+':
				    stk.push(num); break;
				case '-':
				    stk.push(-num); break;
				case '*':
				    pre = stk.top();
				    stk.pop();
				    stk.push(pre * num);
				    break;
				case '/':
				    pre = stk.top();
				    stk.pop();
				    stk.push(pre / num);
				    break;
			    }
			    sign=c;
			    num=0;
			}
			if(c==')'){
			    break;
			}
		    }
		    int res = 0;
		    while (!stk.empty()) {
			res += stk.top();
			stk.pop();
		    }
		    return res;
		}
-	**字符串编码**


	
		class Solution
		public:
		    string src; 
		    size_t ptr;
		    int getDigits()
			int ret = 0;
			while (ptr < src.size() && isdigit(src[ptr])) 
			    ret = ret * 10 + src[ptr++] - '0';
			return ret;
		    string getString() {
			if (ptr == src.size() || src[ptr] == ']') 
			    return "";
			char cur = src[ptr]; int repTime = 1;
			string ret;
			if (isdigit(cur)) {
			    repTime = getDigits(); ++ptr;
			    string str = getString(); ++ptr;
			    while (repTime--) ret += str; 
			else if (isalpha(cur)) 
			    ret = string(1, src[ptr++]);
			return ret + getString();
		    string decodeString(string s) 
			src = s;
			ptr = 0;
			return getString();

