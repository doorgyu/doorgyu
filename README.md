#define _CRT_SECURE_NO_WARNINGS
#include <stdio.h>
#include <windows.h>	// Sleep(), System()
#include <stdlib.h>
#include <conio.h>	// kbhit(),getch()
#include <stdbool.h>

#define INSERT 13
#define DEL 15
#define SEARCH 17
#define UPDATE 19
#define VIEW 21
#define DEL_ALL 23
#define SAVE 25
#define EXIT 27

int x, y;		// 전역변수 커서 좌표

typedef struct student {
	char name[10];	// 이름 ( 한글 5자 미만)
	char age[5];	// 나이
	char hakjum[10];	// 특징
	char club[25];	// 사는 지역 ( 한글 12자 미만)
	char call[15];	// 전화번호( 15자 미만)

	struct student* next;	// 다음 사람의 링크를 참조
}student;

student* head, * tail;		// student형 전역 포인터 변수

void menu_display();	// 메뉴를 표시
void Cursor(bool, int);	// 커서 표시를 위함.
void move(int);	// 키보드 입력에 따른 커서 이동
void run(int);	// 각 메뉴에 따라서 동작 함수 결정
void make_student(void);	// 사람 노드 생성
student* insert_student(void);	// 노드 생성 후 삽입

void view();	// 모든 노드를 보여준다.
void update();	// 사람 노드를 수정한다.
void find_name();	// 이름 검색 후 출력
void del();	// 지울 이름을 검색한 후 삭제
void load();	// 파일에서 사람 데이터를 불러옴.
void save();	// 파일에 사람을 저장한다.
void gotoxy(int, int);	// (x,y)좌표로 커서를 이동한다.

int main() {
	char key;		// case 분기를 위한 변수
	x = 3, y = VIEW;	// 메뉴 출력시 "▶"가 있을 위치 설정

	Cursor(false, 1);	// 콘솔 커서 숨기기
	menu_display();		// 메뉴 화면 출력
	make_student();		// 사람 추가
	load();			// 저장했던 파일 불러오기

	while (1) {
		key = _getch();		// 문자 즉시 저장
		switch (key) {
		case 13:		// Enter키 == 13		
			run(y);		// y좌표의 위치에 따라 동작이 바뀜.
			break;
		case 72:		// ↑ == 72 
			move(-2);
			break;
		case 80:		// ↓ == 80
			move(2);
			break;
		case 27:		// ESC키 == 27
			gotoxy(5, 20);
			exit(0);	// 정상 종료
			break;
		default:
			break;
		}
	}
	return 0;
}
void Cursor(bool flag, int size)
{	// 화살표가 커서의 역할을 하므로 지저분한 콘솔 커서를 숨긴다.

	CONSOLE_CURSOR_INFO cursorInfo;
	cursorInfo.dwSize = size;
	cursorInfo.bVisible = flag;
	SetConsoleCursorInfo(GetStdHandle(STD_OUTPUT_HANDLE), &cursorInfo);
}

void menu_display()		// 메뉴 표시
{
	system("cls");		// 화면 청소
	gotoxy(5, 2);	 puts("사람 정보 관리 프로그램입니다.");
	gotoxy(5, 5);	 puts("프로그램 사용법 ☞	이동: 방향키(↑,↓)");
	gotoxy(5, 7);	 puts("			선택: Enter");
	gotoxy(5, 9);	 puts("			종료: Esc");

	gotoxy(5, INSERT);	puts("1.사람 입력");
	gotoxy(5, DEL);		puts("2.사람 삭제");
	gotoxy(5, SEARCH);	puts("3.사람 검색");
	gotoxy(5, UPDATE);	puts("4.사람 수정");
	gotoxy(5, VIEW);	puts("5.전체 사람 출력");
	gotoxy(5, DEL_ALL);	puts("6.전체 사람 삭제");
	gotoxy(5, SAVE);	puts("7.파일에 저장");
	gotoxy(5, EXIT);	puts("8.프로그램 종료");

	gotoxy(2, y);		// 모두 출력한 후, 원래의 커서 위치로 이동
	printf("▶");

}
void move(int q)		// 키보드 입력에 따른 커서 이동(+1,-1: 아래, 위로);
{
	gotoxy(2, y);
	printf("  ");		// 전에 있던 ▶는 공백으로 덮어쓴다.
	gotoxy(2, y = y + q);	// q만큼 y좌표 이동
	if (y > 28) y = 13;	// 13 ~ 27 사이에서만 커서 이동
	if (y < 12) y = 27;
	gotoxy(2, y);

	printf("▶");		// 해당 커서의 위치로 이동 후 ▶를 출력한다.
}

void run(int y)
{
	switch (y) {
	case INSERT:		// "1.사람 입력"에서 엔터키 누르면
		menu_display();
		insert_student();
		break;
	case DEL:		// "2.사람 삭제"에서 엔터키 누르면
		del();
		break;
	case SEARCH:		// "3.사람 검색"에서 엔터키 누르면
		find_name();
		break;
	case UPDATE:		// "4.사람 수정"에서 엔터키 누르면
		update();
		break;
	case VIEW:		// "5.전체 사람 출력"에서 엔터키 누르면
		view();
		break;
	case DEL_ALL:		// "6.전체 사람 삭제"에서 엔터키 누르면
		make_student();	// 모든 노드를 지우기 위해 head와 tail의 링크를 tail로 한다.
		gotoxy(5, 10);
		printf("데이터 전체 삭제됨.\n");
		break;
	case SAVE:		// "7.파일에 저장"에서 엔터키 누르면
		save();
		break;
	case EXIT:		// "8.프로그램 종료"에서 엔터키 누르면
		exit(0);
		break;
	}
	if (y != DEL_ALL) menu_display();
	// 5번으로 인해 출력된 '데이터 전체 삭제됨'을 지우고 새롭게 메뉴 출력
}

void make_student(void)	// 노드 생성
{
	head = (student*)malloc(sizeof(student));
	tail = (student*)malloc(sizeof(student));
	head->next = tail;	// head포인터는 NULL포인터를 가리킴.
	tail->next = tail;	// (처음에는 노드가 하나도 없으므로)
}

student* insert_student(void)	// 노드 생성 후 삽입
{
	student* s;
	s = (student*)malloc(sizeof(student));
	gotoxy(25, INSERT);		puts("이름:");
	gotoxy(35, INSERT);		fgets(s->name, 10, stdin);
	gotoxy(25, INSERT + 2); puts("나이:");
	gotoxy(35, INSERT + 2); fgets(s->age, 5, stdin);
	gotoxy(25, INSERT + 4); puts("특징:");
	gotoxy(35, INSERT + 4); fgets(s->hakjum, 10, stdin);
	gotoxy(25, INSERT + 6);	puts("사는지역:");
	gotoxy(35, INSERT + 6);	fgets(s->club, 25, stdin);
	gotoxy(25, INSERT + 8);	puts("전화번호:");
	gotoxy(39, INSERT + 8); fgets(s->call, 15, stdin);	// 마지막에 개행문자 입력

	s->next = head->next;
	head->next = s;

	return s;
}

void view()	// 모든 노드를 보여준다.
{
	student* s;
	int count = 1;

	s = head->next;	// head포인터가 가리키는 첫 번째 노드를 대입

	if (head->next == tail) {
		gotoxy(18, VIEW);
		puts("...저장된 명함이 없습니다.");
		Sleep(1000);
	}
	system("cls");
	while (s != tail) {
		printf("-----------------------\n");
		printf("%2d. ", count++);
		printf("이    름:%s", s->name);
		printf("    나    이:%s", s->age);
		printf("    특    징:%s", s->hakjum);
		printf("    사는 지역:%s", s->club);
		printf("    전화번호:%s\n", s->call);
		// 뒤에 \n을 하지 않는 이유는 stdin로 입력을 받았으므로
		// 마지막에 자동으로 개행문자가 입력되기 때문이다.
		s = s->next;
	}
	printf("\n\n");
	puts("초기 메뉴로 돌아갑니다. 아무 키나 눌러주세요.");
	_getch();	// 키 입력을 받을 때까지 대기
}

void update() {	// 사람의 정보를 수정한다.
	char name[10];
	student* s;

	system("cls");
	printf("\n\n정보를 수정할 사람의 이름을 입력하세요: ");

	fgets(name, 10, stdin);

	s = head->next;	// head포인터가 가리키는 첫 번째 노드를 대입

	while (1) {
		if (!(strcmp(name, s->name))) {	// 두 문자열이 같을 때 !0으로 참이 된다.
			gotoxy(25, INSERT - 2); puts("사람의 정보를 수정합니다.");

			gotoxy(25, INSERT + 1);	puts("    이름:");
			gotoxy(35, INSERT + 1);	fgets(s->name, 10, stdin);
			gotoxy(25, INSERT + 2); puts("    나이:");
			gotoxy(35, INSERT + 2); fgets(s->age, 5, stdin);
			gotoxy(25, INSERT + 3); puts("    특징:");
			gotoxy(35, INSERT + 3); fgets(s->hakjum, 10, stdin);
			gotoxy(25, INSERT + 4);	puts("    사는지역:");
			gotoxy(35, INSERT + 4);	fgets(s->club, 25, stdin);
			gotoxy(25, INSERT + 5); puts("    전화번호:");
			gotoxy(39, INSERT + 5); fgets(s->call, 15, stdin);

			puts("\n\n          정보가 수정되었습니다.");

			Sleep(2000);	// 2초 대기 후 break
			break;
		}
		if (s == tail) {	// 끝까지 못 찾으면
			puts("\n해당 사람의 정보가 없습니다.\n");
			Sleep(2000);	// 2초 대기 후 break
			break;
		}
		s = s->next;	// 다음 노드를 찾음.
	}
}
void find_name(void)	// 이름 검색 후 출력
{
	char name[10];
	student* s;

	system("cls");
	printf("\n\n찾을 사람의 이름을 입력하세요: ");

	fgets(name, 10, stdin);

	s = head->next;	// head포인터가 가리키는 첫 번째 노드를 대입

	while (1)
	{
		if (!(strcmp(name, s->name))) {	// 두 문자열이 같을 때 !0으로 참이 된다.
			puts("\n사람을 찾았습니다.");
			puts("-----------------------------");
			printf("◈ 이   름  : %s", s->name);
			printf("§ 나   이  : %s", s->age);
			printf("Α 특   징  : %s", s->hakjum);
			printf("㈜ 사는지역  : %s", s->club);
			printf("☎ 전화번호 : %s", s->call);
			puts("-----------------------------\n");
			break;
		}
		if (s == tail) {
			puts("\n찾는 이름이 없습니다.\n");
			break;
		}
		s = s->next;
	}
	puts("계속 찾으시려면 @, 이전으로 돌아가시려면 아무키나 눌러주세요.");
	if (_getch() == '@') find_name();	// @를 누르면 반복
}

void del(void)		// 지울 이름을 검색한 후 삭제
{
	char name[10];
	student* a, * b;

	a = head;
	b = a->next;

	system("cls");
	printf("지울 이름을 입력하세요: ");
	fgets(name, 10, stdin);

	while (1)
	{
		if (!(strcmp(name, b->name)))
		{
			a->next = b->next;
			free(b);
			printf("\n 해당 사람을 지웠습니다 ->%s", name);
			Sleep(1000);		// 1000분의 1초 단위로 정지
			break;
		}
		if (b == tail)
		{
			puts("\n찾는 이름이 없습니다.\n");
			break;
		}
		a = b;
		b = a->next;
	}
	puts("계속 삭제하려면 @, 이전으로 가려면 아무키나 눌러주세요.");
	if (_getch() == '@') del();
	// @를 누르면 반복
}

void load(void)			// 파일에서 사람 데이터를 불러옴.
{
	student* a, * b;
	FILE* fp;

	a = head;
	b = a->next;
	while (b != tail)
	{
		a = b;
		b = a->next;		// 기존의 사람 데이터를 다 지움.
		free(a);
	}
	head->next = tail;

	if ((fp = fopen("student.dat", "rb")) == NULL) return;
	// "student.dat"에서 파일을 읽어와서 fp에 저장한다. NULL이면 함수 종료

	while (1)	// NULL이 아니라면
	{
		a = (student*)malloc(sizeof(student));
		if (!fread(a, 61, 1, fp))// 구조체 멤버들의 byte크기만큼 a에 읽어들임.
		{
			free(a);
			break;		// 읽어들일 값이 없다면 break;
		}
		a->next = head->next;
		head->next = a;
	}
	fclose(fp);
}

void save(void)	// 파일에 사람 저장
{
	FILE* fp;
	student* a;

	fp = fopen("student.dat", "wb");
	if (fp == NULL)puts("저장할 파일을 열 수 없네요."), exit(1);

	a = head->next;
	while (a != tail)
	{
		fwrite(a, 61, 1, fp);// 구조체 멤버들의 byte크기만큼 기록한다.
		a = a->next;
	}
	fclose(fp);
	gotoxy(18, SAVE);
	puts("....데이터 파일로 저장하였습니다.");
	Sleep(1000);

}

void gotoxy(int x, int y)	// (x,y)좌표로 커서를 이동
{
	COORD pos = { x - 1,y - 1 };	// 원점은 (0,0)
	SetConsoleCursorPosition(GetStdHandle(STD_OUTPUT_HANDLE), pos);
}
