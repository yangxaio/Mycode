# Mycode
#include<iostream>
#include<string>
using namespace std;
#define max_free 10    //空闲区最大个数
#define max_job 10   //已分分区最大个数

struct table {
	float address; //起始地址
	float length;  //分区长度
	string flag;
	//0表示空栏目，1表示未分配,作业名则表示分配给某作业。
}free_table[max_free], used_table[max_job];
static float Size = 0;    //最小切割长度

void init(table free_table[], table used_table[]) {
	/*初始化空闲分区*/
	for (int i = 0; i < max_free; i++) {
		free_table[i].flag = "0";
		used_table[i].flag = "0";
	}
	int s;   //初始空闲分区个数
	printf("请输入初始空闲分区的个数：\n");
	scanf_s("%d", &s);
	for (int i = 0; i < s; i++) {
		printf("输入第%d块空闲分区始址、大小：\n", i + 1);
		scanf_s("%f%f", &free_table[i].address, &free_table[i].length);
		free_table[i].flag = "1";
	}
	printf("请输入最小可分割长度：\n");
	scanf_s("%f", &Size);
	printf("\n\n");
}

void sort(table free_table[]) {
	/*对空闲区按照内存大小进行排序*/
	for (int i = 0; i < max_free - 1; i++) {
		if (free_table[i].length > free_table[i + 1].length) {
			float tmp;   tmp = free_table[i].address;  free_table[i].address = free_table[i + 1].address;  free_table[i + 1].address = tmp;
			float temp;  temp = free_table[i].length;  free_table[i].length = free_table[i + 1].length;  free_table[i + 1].length = temp;
			string tan;   tan = free_table[i].flag;   free_table[i].flag = free_table[i + 1].flag;   free_table[i + 1].flag = tan;
		}
	}
}
void BF(table free_table[], table used_table[]) {
	/*最佳适应算法进行内存分配*/
	printf("请输入作业名称、内存大小：\n");
	float length;   //该作业内存大小
	string name;
	cin >> name >> length;
	//对空闲区，按照空闲区从小到大进行排序。
	sort(free_table);
	bool k = false;  //判断是否找到空闲区
	for (int i = 0; i < max_free; i++) {
		if (free_table[i].flag == "1" && free_table[i].length >= length) {
			if (free_table[i].length - length < Size) {
				//空闲区不进行切割
				free_table[i].flag = name;   //空闲区状态设置为作业占用
				used_table[i].address = free_table[i].address;
				used_table[i].length = free_table[i].length;
				used_table[i].flag = name;
				k = true;
				break;
			}
			else {
				//空闲区进行切割
				used_table[i].address = free_table[i].address;
				used_table[i].length = length;
				used_table[i].flag = name;
				free_table[i].address += length;
				free_table[i].length -= length;
				k = true;
				break; 
			}
		}
	}
	if (k == false)
		printf("无可用空闲区！\n");
	sort(free_table);
	printf("\n\n");
}

void Recycle(table free_table[], table used_table[]) {
	printf("请输入内存回收的作业名称：\n");
	string name;
	cin >> name;
	bool k = false;   //判断已分配分区中是否有该作业
	bool tag = false;  //判断是否有相邻空闲区
	for (int i = 0; i < max_job; i++) {
		if (used_table[i].flag == name) {
			for (int j = 0; j < max_free; j++) {
				/*判断作业两旁是否有空闲区*/
				if ((free_table[j].address + free_table[j].length) == used_table[i].address && free_table[j].flag == "1") {
					/*作业上方有空闲区*/
					used_table[i].flag = "0";  //将该分区置为空栏目
					free_table[j].length += used_table[i].length;
					tag = true;
					break;
				}
				if ((used_table[i].address + used_table[i].length) == free_table[j].address && free_table[j].flag == "1") {
					/*作业下方有空闲区*/
					used_table[i].flag = "0";
					free_table[j].address -= used_table[i].length;
					free_table[j].length += used_table[i].length;
					tag = true;
					break;
				}
				for (int m = j + 1; m < max_free; m++) {
					if ((free_table[j].address + free_table[j].length) == used_table[i].address 
						&& (used_table[i].address + used_table[i].length) == free_table[m].address && free_table[j].flag == "1") {
						/*作业上下都有空闲区*/
						used_table[i].flag = "0";
						free_table[m].flag = "0";
						free_table[j].length = used_table[i].length + free_table[m].length;
						tag = true;
						break;
					}
				} //end of for
			}   //end of for
		    /*没有空闲区合并情况*/
			if (tag == false) {  //tag判断是否有相邻空闲区
				for (int n = 0; n < max_free; n++) {
					if (free_table[n].flag == "0") {
						used_table[i].flag = "0";
						free_table[n].address = used_table[i].address;
						free_table[n].length = used_table[i].length;
						free_table[n].flag = "1";
						break;
					}
				}
			}
			k = true;   //标志已分配分区中有该作业
			break;  //如果存在，找到并合并完即可，无需再向下循环，提高效率
		} //end of if
	}  //end of for
	if (k == false) {
		printf("未找到作业%s，请确定后重新输入！\n", name.c_str());
		Recycle(free_table, used_table);
	}
	sort(free_table);
	printf("\n");
}

void Display_free(table free_table[]) {
	printf("空闲区：\n");
	printf("始址\t大小\t当前状态\n");
	for (int i = 0; i < max_free; i++) {
		if (free_table[i].flag == "1") {
			string state = "未分配";
			printf("%.0f\t%.0f\t%s\n",free_table[i].address,free_table[i].length,state.c_str());
		}
	}
	printf("\n\n");
}

void Display_used(table used_table[]) {
	printf("已分配分区:\n");
	printf("始址\t大小\t当前状态\n");
	for (int i = 0; i < max_job; i++) {
		if (used_table[i].flag != "0" && used_table[i].flag != "1") 
			printf("%.0f\t%.0f\t%s\n", used_table[i].address, used_table[i].length, used_table[i].flag.c_str());
	}
	printf("\n\n");
}

void chose(table free_table[], table used_table[]) {
	while (true) {
		int q;
		printf("请输入操作对应数字完成操作！\n");
		printf("1、首次适应算法分配内存\t2、回收作业内存\t3、空闲区信息\n\n4、已分配分区信息\t5、空闲分区初始化\t6、退出\n");
		scanf_s("%d", &q);
		switch (q) {
		case 1:
			BF(free_table, used_table);
			break;
		case 2:
			Recycle(free_table, used_table);
			break;
		case 3:
			Display_free(free_table);
			break;
		case 4:
			Display_used(used_table);
			break;
		case 5:
			init(free_table, used_table);
			break;
		case 6:
			exit(0);  //正常退出程序
			break;
		}
	}
}
void main() {
	init(free_table, used_table);
	chose(free_table, used_table);
}
