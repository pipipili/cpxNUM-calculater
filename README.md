# cpxNUM-calculater
//编程实践1：复数计算器及窗口设计
#include <windows.h>
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#define                BTN0                100
#define                BTN1                101
#define                BTN2                102
#define                BTN3                103
#define                BTN4                104
#define                BTN5                105
#define                BTN6                106
#define                BTN7                107
#define                BTN8                108
#define                BTN9                109//0到9
#define                BTN_PIT             110//.
#define                BTN_ADD                112
#define                BTN_SUB                113
#define                BTN_MUL                114
#define                BTN_DIV                115
#define                BTN_EQU                116//=
#define                BTN_CLS                117//清空全部进程
#define                BTN_NXT                118//输入下一个复数
#define                BTN_OPR                119//输入对两复数的操作
#define                TEXT_BOX        120

typedef struct{
double real;//实部
double imag;//虚部
}cpxNUM;//抽象类型：复数

void create_button(HWND hwnd,int x, int y,int id);
void create_textbox(HWND hwnd);
void onCommand(HWND hwnd,WPARAM wParam);
void print(cpxNUM c);
LRESULT CALLBACK WndProc(HWND,UINT, WPARAM, LPARAM);
cpxNUM cplus(cpxNUM c1,cpxNUM c2);
cpxNUM cmilus(cpxNUM c1,cpxNUM c2);
cpxNUM cmultiply(cpxNUM c1,cpxNUM c2);
cpxNUM cdivide(cpxNUM c1,cpxNUM c2);
cpxNUM num1,num2;

char string[50],cz=0;
int count=1,judge=0;
HINSTANCE ghInstance;

char labels[][19]= { "1", "2", "3", "4", "5",
                     "6", "7", "8", "9", "0",
                     "+", "-", "*", "/", "=",
                     ".","CLR","NXT","OPR"
                     };
int bid[] = {
        BTN1, BTN2, BTN3, BTN4, BTN5,
        BTN6, BTN7, BTN8, BTN9, BTN0,
        BTN_ADD, BTN_SUB, BTN_MUL, BTN_DIV, BTN_EQU,
        BTN_PIT, BTN_CLS, BTN_NXT, BTN_OPR
        };

INT WINAPI WinMain(HINSTANCE hInstance,HINSTANCE hPrevInstance,PSTR str,INT iCmdShow) {//主函数
    HWND hWnd;
    MSG msg;
    WNDCLASS wndClass;

    wndClass.style = CS_HREDRAW | CS_VREDRAW;//窗口样式
    wndClass.lpfnWndProc = WndProc;//窗口处理函数
    wndClass.cbClsExtra = 0;//窗口类扩展：无
    wndClass.cbWndExtra = 0;//窗口实例扩展：无
    wndClass.hInstance = hInstance;//窗口实例句柄
    wndClass.hIcon = LoadIcon(NULL,IDI_APPLICATION);//窗口最小化图标：使用缺省图标
    wndClass.hCursor = LoadCursor(NULL,IDC_ARROW);//窗口采用箭头光标
    wndClass.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);//窗口背景颜色：白色
    wndClass.lpszMenuName = NULL;//窗口菜单：无
    wndClass.lpszClassName = TEXT("calc");//窗口类名：计算器

    if(!RegisterClass(&wndClass)){//注册窗口类，失败则弹出提醒
        MessageBox(NULL,TEXT("fail to register the window"),TEXT("ERROR"),MB_OK);
        return 0;
    }

    ghInstance = hInstance;

    hWnd = CreateWindow(//创建窗口
        TEXT("calc"),   // 窗口类名
        TEXT("calculator-noxue.com"),// 窗口标题
        WS_OVERLAPPEDWINDOW,//窗口风格
        CW_USEDEFAULT,//窗口初始显示位置x：使用缺省值
        CW_USEDEFAULT,//窗口初始显示位置y：使用缺省值
        210,//窗口的宽度
        230,//窗口的高度
        NULL,//父窗口：无
        NULL,//子菜单：无
        hInstance,//该窗口应用程序的实例句柄
        NULL//
        );

    ShowWindow(hWnd, iCmdShow);//显示窗口
    UpdateWindow(hWnd);//更新窗口

    while (GetMessage(&msg,NULL, 0, 0)) {//从消息列队中获取消息
        TranslateMessage(&msg);//将虚拟键消息转化为字符消息
        DispatchMessage(&msg);//分发到回调函数
    }
    return msg.wParam;
}

LRESULT CALLBACK WndProc(HWND hWnd,UINT message,WPARAM wParam,LPARAM lParam) {
    HDC hdc;//设置环境句柄
    PAINTSTRUCT ps;//绘制结构

    switch (message) {//处理得到的消息
        case WM_CREATE:{//窗口创建完成时得到的消息
            for (int i = 0; i < 19; i++) {//创建按钮
                create_button(hWnd, 10 + (i % 5 * 35), 50 + (i / 5 * 35), i);//每个按钮的位置
            }
            create_textbox(hWnd);
            return 0;
        }
        case WM_COMMAND:{//点击按钮时得到的消息
            onCommand(hWnd, wParam);//执行计算命令
            return 0;
        }
        case WM_PAINT:{//改变窗口时得到的消息
            hdc = BeginPaint(hWnd, &ps);
            EndPaint(hWnd, &ps);
            return 0;
        }
        case WM_DESTROY:{//窗口关闭时的消息
            PostQuitMessage(0);
            return 0;
        }
        default:
            return DefWindowProc(hWnd,message, wParam, lParam);
    }
} // WndProc

void create_button(HWND hwnd,int x,int y,int id) {//创建按钮
    CreateWindow("Button", labels[id],WS_CHILD | WS_VISIBLE | BS_PUSHBUTTON, x, y, 30, 30,
            hwnd, (HMENU)bid[id], ghInstance,NULL);
}

void create_textbox(HWND hwnd) {//创建编辑框
    CreateWindow("Edit",NULL, WS_CHILD | WS_VISIBLE | WS_BORDER | WS_DISABLED, 10, 10, 170,
            30,hwnd, (HMENU)TEXT_BOX, ghInstance,NULL);
}

void onCommand(HWND hwnd,WPARAM wParam) {
    WORD w = LOWORD(wParam);
    HWND h = NULL;
    char buf[50] = "", text[100] = "";

    if ((w >= BTN0 && w <= BTN_PIT)||(w==BTN_SUB&&!judge)) {//如果是0-9、小数点以及-被按下，表示正在输入
        h = GetDlgItem(hwnd, w);//获取被单击的子控件的句柄
        GetWindowText(h, buf, 50);//获取被单击的子控件的内容，将h中的标题行文本拷贝到缓冲区buf中，50为缓冲区长度
        h = GetDlgItem(hwnd, TEXT_BOX);//获取编辑框的句柄
        GetWindowText(h, text, 500);//获取编辑框的内容
        strcat(text, buf);//把内容拼接起来
        SetWindowText(h, text);//设置编辑框内容
    }

    else if (w==BTN_NXT){//输入下一个数值
        h = GetDlgItem(hwnd, TEXT_BOX);//获取编辑框句柄
        memset(text, 0, 100);//将first后的100个字节用0代替
        GetWindowText(h,text,500);//将编辑框中内容拷贝到text
        switch (count){//根据计数器count的数值，将输入的数按序储存
            case 1:{
                num1.real=atol(text);
                break;
            }
            case 2:{
                num1.imag=atol(text);
                break;
            }
            case 3:{
                num2.real=atol(text);
                break;
            }
        }
        count+=1;
        SetWindowText(h, "");//清空编辑框
    }

    else if (w==BTN_OPR) {//此使四个数(两实部两虚部)均已输入
        h = GetDlgItem(hwnd, TEXT_BOX);
        GetWindowText(h,text,500);
        num2.imag=atol(text);
        SetWindowText(h,"");
        judge=count=1;
    }

    else if (w>=BTN_ADD&&w<=BTN_DIV&&judge){//输入对两复数的操作
        h = GetDlgItem(hwnd, w);//保存哪个操作按钮(+-*/)被单击
        GetWindowText(h, buf, 50);
        cz = buf[0];
        h = GetDlgItem(hwnd, TEXT_BOX);
        SetWindowText(h,buf);
    }

    else if (w ==BTN_EQU) {
        h = GetDlgItem(hwnd, TEXT_BOX);
        SetWindowText(h,"");
        cpxNUM result;
        result.real=result.imag=0;
        switch (cz) {
            case '+':{
                result = cplus(num1,num2);
                break;
            }
            case '-':{
                result = cmilus(num1,num2);
                break;
            }
            case '*':{
                result = cmultiply(num1,num2);
                break;
            }
            case '/':{
                result = cdivide(num1,num2);
                break;
            }
            default:
                break;
        }
        print(result);
        sprintf(text, "%s", string);
        SetWindowText(h, text);
        judge=0;
    }

    else if (w ==BTN_CLS) {
        h = GetDlgItem(hwnd, TEXT_BOX);
        SetWindowText(h, "");
        num1.real=num1.imag=num2.real=num2.imag=judge=0;
        count=1;
    }
}

cpxNUM cplus(cpxNUM c1,cpxNUM c2){//加法，返回值为和
    cpxNUM result;
    result.real=c1.real+c2.real;
    result.imag=c1.imag+c2.imag;
    return result;
}

cpxNUM cmilus(cpxNUM c1,cpxNUM c2){//减法，返回值为差
    cpxNUM result;
    result.real=c1.real-c2.real;
    result.imag=c1.imag-c2.imag;
    return result;
}

cpxNUM cmultiply(cpxNUM c1,cpxNUM c2){//乘法，返回值为积
    cpxNUM result;
    result.real=c1.real*c2.real-c1.imag*c2.imag;
    result.imag=c1.real*c2.imag+c1.imag*c2.real;
    return result;
}

cpxNUM cdivide(cpxNUM c1,cpxNUM c2){//除法，返回值为商
    cpxNUM result;
    result.real=(c1.real*c2.real-c1.imag*c2.imag)/(c2.real*c2.real+c2.imag*c2.imag);
    result.imag=(c1.imag*c2.real-c1.real*c2.imag)/(c2.real*c2.real+c2.imag*c2.imag);
    return result;
}

void print(cpxNUM c){//以复数形式输出
    char str2[25];
    if(c.real){
        sprintf(string,"%lf",c.real);
        if(c.imag<0)
            sprintf(str2,"%lfi",c.imag);
        else if(c.imag>0)
            sprintf(str2,"+%lfi",c.imag);
        else;
        strcat(string,str2);
    }
    else{
        if(!c.imag)
            sprintf(string,"0");
        else
            sprintf(string,"%lfi",c.imag);
    }
}
