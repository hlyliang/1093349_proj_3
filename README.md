# 目錄
- [題目](#題目)
- [輸入輸出範例](#輸入輸出範例)
  - [input](#input)
  - [output](#output)
- [說明文件](#說明文件)
  - [struct](#struct)
  - [全域變數](#全域變數)
  - [main function](#main)
  - [issue function](#issue)
  - [execute function](#execute)
  - [wr function](#wr)
  - [print function](#print)
  

# 題目
Tomasulo algorithm 的過程

***
# 輸入輸出範例
- #### input
![input](https://user-images.githubusercontent.com/103658997/174035279-55d35cf2-a734-473d-b6ee-e4c6fa88e568.png)

- #### output
![output1](https://user-images.githubusercontent.com/103658997/174035340-7b4c4d7d-7b7c-431a-a15e-2c683f6d4782.png) 
![output6](https://user-images.githubusercontent.com/103658997/174035346-f045a255-f011-428b-9fb1-cc887dad1762.png) 
![output45](https://user-images.githubusercontent.com/103658997/174035347-91208b10-607c-4c7c-8006-fd856aa74c75.png) 
![output64](https://user-images.githubusercontent.com/103658997/174035350-e2d05715-5c9c-4907-8f1c-940e9345de92.png) 

***
# 說明文件
- #### struct
##### 建立 struct 存 rs 及 buffer 內的值
```c++
struct RS
{
	bool use = false;
	string sign;
	string reg;
	string n1;
	string n2;
	int rsnum;
	int cycle_time;
};
```
***
- #### 全域變數
```c++
int cycle = 1; 
vector<string>input_temp[8]; 
RS buffer[2]; 
RS add_rs[3];
RS mul_rs[2];
map<string, int> rf = { {"F1",0},{"F2",2},{"F3",4},{"F4",6},{"F5",8} };
map<string, string> rat = { {"F1",""},{"F2",""},{"F3",""},{"F4",""},{"F5",""} };
bool add_exe = false;
bool mul_exe = false;
bool change = false;
```

***
- #### main
  - ##### 讀取使用著輸入的 code 並做分割
  - ##### 判斷當前的 cycle 能不能做 issue、execute、write result
  - ##### 有做的話就要把改變的結果 print 出來
```c++
int main()
{
	string temp;
	vector<string> input;	
	string space_delimiter1 = " ";
	string space_delimiter2 = ",";
	

	cout << endl << "please input code :" << endl;
	while (getline(cin, temp) && temp != "")
	{
		input.push_back(temp);
	}

	for (int i = 0; i < input.size(); i++)
	{
		size_t pos = 0;
		while ((pos = input[i].find(space_delimiter1)) != string::npos)
		{
			if (input[i].substr(0, pos).find(space_delimiter2) != string::npos)
				input_temp[i].push_back(input[i].substr(0, pos-1));
			else
				input_temp[i].push_back(input[i].substr(0, pos));			
			input[i].erase(0, pos + space_delimiter1.length());

		}
		input_temp[i].push_back(input[i]);		
	}

	int in = 0, wn = 0, n = 0;
	while (wn < input.size())
	{
		if (cycle == buffer[0].cycle_time)
		{
			wr(0);
			wn++;
		}
		if (cycle == buffer[1].cycle_time)
		{
			wr(1);
			wn++;
		}

		if (add_exe == false)
		{
			for (int i = 0; i < 3; i++)
			{
				if (add_rs[i].use == true)
				{
					if (add_rs[i].n1[0] != 'R' && add_rs[i].n2[0] != 'R')
					{
						add_exe = true;
						execute(i, 0);
					}
				}
			}
		}
		if (mul_exe == false)
		{
			for (int j = 0; j < 2; j++)
			{
				if (mul_rs[j].use == true)
				{
					if (mul_rs[j].n1[0] != 'R' && mul_rs[j].n2[0] != 'R')
					{
						mul_exe = true;
						execute(j, 1);
					}
				}
			}
		}

		while (in < input.size())
		{
			if (input_temp[in].size()) 
			{
				if (input_temp[in][0] == "ADDI" || input_temp[in][0] == "ADD" || input_temp[in][0] == "SUB")
				{
					for (n = 0; n < 3; n++)
					{
						if (add_rs[n].use == false)
						{
							issue(input_temp[in],n);
							add_rs[n].use = true;
							input_temp[in].clear();
							break;
						}

					}
				}	
				else
				{
					for (n = 0; n < 2; n++)
					{
						if (mul_rs[n].use == false)
						{
							issue(input_temp[in],n);
							mul_rs[n].use = true;
							input_temp[in].clear();
							break;
						}
					}
				}
				
				break;
			}				
			else in++;
		}
		
		if (change == true)
		{
			print();
			change = false;
		}
		cycle++;
	}
}
```
***
- #### issue
  - ##### 判斷是哪種 case 並把值放入 rs 中 
```c++
void issue(vector<string> input, int i)
{
	if (input[0] == "ADDI" || input[0] == "ADD" || input[0] == "SUB")
	{
		add_rs[i].use = true;
		if (input[0] == "SUB")
			add_rs[i].sign = "-";
		else add_rs[i].sign = "+";

		if (rat[input[2]] == "") // rat is empty
			add_rs[i].n1 = to_string(rf[input[2]]);
		else add_rs[i].n1 = rat[input[2]];

		if (input[0] == "ADDI")
			add_rs[i].n2 = input[3];
		else
		{
			if (rat[input[3]] == "")
				add_rs[i].n2 = to_string(rf[input[3]]);
			else add_rs[i].n2 = rat[input[3]];
		}

		add_rs[i].reg = input[1];
		rat[input[1]] = "RS" + to_string(i + 1);
	}
	
	if (input[0] == "MUL" || input[0] == "DIV")
	{

		if (input[0] == "MUL")
			mul_rs[i].sign = "*";
		else mul_rs[i].sign = "/";

		if (rat[input[2]] == "") // rat is empty
			mul_rs[i].n1 = to_string(rf[input[2]]);
		else mul_rs[i].n1 = rat[input[2]];

		if (rat[input[3]] == "") // rat is empty
			mul_rs[i].n2 = to_string(rf[input[3]]);
		else mul_rs[i].n2 = rat[input[3]];

		mul_rs[i].reg = input[1];
		rat[input[1]] = "RS" + to_string(i + 4);
	}
	change = true;
}
```
***
- #### execute
  - ##### 把 rs 的值放入 buffer 中並記錄甚麼時候會做完
```c++
void execute(int n, int bn)
{
	change = true;
	if (bn == 0)
	{
		buffer[bn] = add_rs[n];
		buffer[bn].cycle_time = cycle + 2;
	}
	else
	{
		buffer[bn] = mul_rs[n];
		if (buffer[bn].sign == "*")
			buffer[bn].cycle_time = cycle + 10;
		else if (buffer[bn].sign == "/")
			buffer[bn].cycle_time = cycle + 40;
	}

	buffer[bn].rsnum = n;	
}
```
***
- #### wr
  - ##### 把值算出來並寫回
```c++
void wr(int bn)
{
	change = true;
	int ans;
	if (bn == 0)
	{
		if (buffer[bn].sign == "+")
			ans = stoi(buffer[bn].n1) + stoi(buffer[bn].n2);
		else ans = stoi(buffer[bn].n1) - stoi(buffer[bn].n2);


		for (int i = 0; i < 3; i++)
		{
			if (add_rs[i].n1 == "RS" + to_string(buffer[bn].rsnum + 1))
				add_rs[i].n1 = to_string(ans);
			if (add_rs[i].n2 == "RS" + to_string(buffer[bn].rsnum + 1))
				add_rs[i].n2 = to_string(ans);
		}
		for (int j = 0; j < 2; j++)
		{
			if (mul_rs[j].n1 == "RS" + to_string(buffer[bn].rsnum + 1))
				mul_rs[j].n1 = to_string(ans);
			if (mul_rs[j].n2 == "RS" + to_string(buffer[bn].rsnum + 1))
				mul_rs[j].n2 = to_string(ans);
		}

		add_rs[buffer[bn].rsnum].cycle_time = 0;
		add_rs[buffer[bn].rsnum].n1 = "";
		add_rs[buffer[bn].rsnum].n2 = "";
		add_rs[buffer[bn].rsnum].reg = "";
		add_rs[buffer[bn].rsnum].rsnum = 0;
		add_rs[buffer[bn].rsnum].sign = "";
		add_rs[buffer[bn].rsnum].use = false;
		add_exe = false;

		if (rat[buffer[bn].reg] == "RS" + to_string(buffer[bn].rsnum + 1))
		{
			rf[buffer[bn].reg] = ans;
			rat[buffer[bn].reg] = "";
		}
	}
	if (bn == 1)
	{
		if(buffer[bn].sign == "*")
			ans = stoi(buffer[bn].n1) * stoi(buffer[bn].n2);
		else ans = stoi(buffer[bn].n1) / stoi(buffer[bn].n2);

		for (int i = 0; i < 3; i++)
		{
			if (add_rs[i].n1 == "RS" + to_string(buffer[bn].rsnum + 4))
				add_rs[i].n1 = to_string(ans);
			if (add_rs[i].n2 == "RS" + to_string(buffer[bn].rsnum + 4))
				add_rs[i].n2 = to_string(ans);
		}
		for (int j = 0; j < 2; j++)
		{
			if (mul_rs[j].n1 == "RS" + to_string(buffer[bn].rsnum + 4))
				mul_rs[j].n1 = to_string(ans);
			if (mul_rs[j].n2 == "RS" + to_string(buffer[bn].rsnum + 4))
				mul_rs[j].n2 = to_string(ans);
		}

		mul_rs[buffer[bn].rsnum].cycle_time = 0;
		mul_rs[buffer[bn].rsnum].n1 = "";
		mul_rs[buffer[bn].rsnum].n2 = "";
		mul_rs[buffer[bn].rsnum].reg = "";
		mul_rs[buffer[bn].rsnum].rsnum = 0;
		mul_rs[buffer[bn].rsnum].sign = "";
		mul_rs[buffer[bn].rsnum].use = false;
		mul_exe = false;

		if (rat[buffer[bn].reg] == "RS" + to_string(buffer[bn].rsnum + 4))
		{
			rf[buffer[bn].reg] = ans;
			rat[buffer[bn].reg] = "";
		}
	}	
	buffer[bn].cycle_time = 0;
	buffer[bn].n1 = "";
	buffer[bn].n2 = "";
	buffer[bn].reg = "";
	buffer[bn].rsnum = 0;
	buffer[bn].sign = "";
	buffer[bn].use = false;
}
```
***
- #### print
  - ##### 把結果印出來
```c++
void print()
{
	cout << "cycle : " << cycle << endl;

	cout << "   __RF__   " << endl;
	for (int i = 1; i < 6; i++)
	{
		cout << "F" << i << " |" << setw(4) << rf["F"+to_string(i)] << " |" << endl;
	}
	cout << "   _______\n\n";

	cout << "   __RAT__" << endl;
	for (int j = 1; j < 6; j++)
	{
		cout << "F" << j << " |" << setw(4) << rat["F" + to_string(j)] << " |" << endl;
	}
	cout << "   _______\n\n";

	cout << "    ________RS________" << endl;
	for (int k = 0; k < 3; k++)
	{
		cout << "RS" << k + 1 << " |" << setw(4) << add_rs[k].sign << " |" << setw(4) << add_rs[k].n1 << " |" << setw(4) << add_rs[k].n2 << " |\n";
	}
	cout << "    __________________\n\n";
	cout << "BUFFER : ";
	if (buffer[0].use == false)
		cout << " empty\n\n";
	else cout << " ( RS" << buffer[0].rsnum+1 << " ) " << buffer[0].n1 << " " << buffer[0].sign << " " << buffer[0].n2 << endl << endl;


	cout << "    ________RS________" << endl;
	for (int l = 0; l < 2; l++)
	{
		cout << "RS" << l + 4 << " |" << setw(4) << mul_rs[l].sign << " |" << setw(4) << mul_rs[l].n1 << " |" << setw(4) << mul_rs[l].n2 << " |\n";
	}
	cout << "    __________________\n\n";
	cout << "BUFFER : ";
	if (buffer[1].use == false)
		cout << " empty\n\n";
	else cout << " ( RS" << buffer[1].rsnum+4 << " ) " << buffer[1].n1 << " " << buffer[1].sign << " " << buffer[1].n2 << endl << endl;
	cout << "--------------------------------------------------------------\n";
}
```
