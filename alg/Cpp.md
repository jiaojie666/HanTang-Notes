<br>

<div align="center">
    <img src="logo.jpg" width="200px">
</div>

<br>

(1)	Memcpy||有重叠和无重叠两种情况


	
	无重叠情况：
	void *memcpy(void *dst, void*source, size_t c)
		void *ans=dst;
		while(c-->=0)
			*((char*)dst++)=	*((char*)source++)	
	有重叠情况，要倒着遍历：
		void *memmove(void*dst,void*source,size_t c)
			void *ans=dst;
			if(dst<=src||(char*)dst>=((char*)src+count))
				同上
			else
				dst=(char*)dst+count-1;
				src=(char*)src+count-1;
				while(c--)
					*((char*)dst--)=	*((char*)source--)
(2)	memset



	void *memset(void* dst,int val, size_t count)
		void *ans=dst;
		while(count--)
			*(char*)dst=(char)val;
			dst=(char*)dst+1; 
		return ans;
(3)	strcpy



	char *strcpy(char* des, const char*src)
			char *address=des;
			while((*des++=*src++)!=’\0’)
			return add

	char *strcat(char*dst,const char*src)
			char *add=dst;
			while(*dst!=’\0’)
				++dst;
			while(*dst++=*src++)
				;
			return add;

	int strcmp(const char* s1,const char* s2)
			while(*s1==*s2)
				if(*s1==’\0’)
					return 0;
				++s1;
				++s2;
	return *s1-*s2;
