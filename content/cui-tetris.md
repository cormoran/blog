+++
date = "2014-10-19T01:28:39+09:00"
title = "cui tetris"

+++

![tetris](https://lh3.googleusercontent.com/-_2UGfEwN0VA/VixfcWQdZ7I/AAAAAAAAAII/Y0fiZ3Yhqkc/s800-Ic42/tetris-300x191.png)

何か簡単なゲームが必要だったのでテトリスもどきを作ってみました。ncursesを使ってます。

<!--more-->

下がプログラムです。最低限の機能しかつけていませんがちょっと楽しいです。まあ、emacsにtetrisは入っているのでそれで遊べばいいのですが。

~~~cpp

#include <ncurses.h>
#include <unistd.h>
#include<cstdio>
#include<cstdlib>
#include<ctime>
#include <utility>
#include <string>
        using namespace std;

#define TIMEOUT 10 //キーボード入力タイムアウト

#define NEXT_BLOCK_X 13
#define NEXT_BLOCK_Y 0

#define SCORE_X 13
#define SCORE_Y 20

#define MAP_X (10+2) //文字　壁2つ
#define MAP_Y (20+1+5) //行　下ガード＋未表示領域
#define MAP_NOT_SHOW_NUM 5
#define BLOCK_H_W 5
#define BLOCK_NUM 7
bool block_can_rot[BLOCK_NUM]={0,1,1,1,1,1,1};
char block_data[BLOCK_NUM][5][5]=
{
    {{0,0,0,0,0},
     {0,2,2,0,0},
     {0,2,2,0,0},
     {0,0,0,0,0},
     {0,0,0,0,0}
    },
    {{0,0,0,0,0},
     {0,0,3,0,0},
     {0,3,3,3,0},
     {0,0,0,0,0},
     {0,0,0,0,0}
    },
    {{0,0,0,0,0},
     {0,0,0,4,0},
     {0,0,4,4,0},
     {0,0,4,0,0},
     {0,0,0,0,0}
    },
    {{0,0,5,0,0},
     {0,0,5,0,0},
     {0,0,5,0,0},
     {0,0,5,0,0},
     {0,0,0,0,0}
    },
    {{0,0,0,0,0},
     {0,6,0,0,0},
     {0,6,6,0,0},
     {0,0,6,0,0},
     {0,0,0,0,0}
    },
    {{0,0,0,0,0},
     {0,0,7,7,0},
     {0,0,7,0,0},
     {0,0,7,0,0},
     {0,0,0,0,0}
    },
    {{0,0,0,0,0},
     {0,8,8,0,0},
     {0,0,8,0,0},
     {0,0,8,0,0},
     {0,0,0,0,0}
    },
};

enum Direction{UP,DOWN,RIGHT,LEFT};

typedef struct block
{
    char data[BLOCK_H_W][BLOCK_H_W];
    bool can_rot;
    pair<int,int> point;
    block()
    {
        reset(0);
    }
    void reset(char type)
    {
        char x,y;
        for(y=0;y<BLOCK_H_W;y++)
            for(x=0;x<BLOCK_H_W;x++)
                data[y][x]=block_data[type][y][x];
        can_rot=block_can_rot[type];
        point.first=3;
        point.second=0;
    }
    bool rotate(Direction dir,char map[MAP_Y][MAP_X])
    {
        if(can_rot){
            char next_data[BLOCK_H_W][BLOCK_H_W];
            //取り敢えず回転させてみる
            if(dir==RIGHT)//右移転
            {
                char x,y;
                for(y=0;y<BLOCK_H_W;y++)
                    for(x=0;x<BLOCK_H_W;x++)
                        next_data[x][(BLOCK_H_W-1)-y]=data[y][x];
            }
            else if(dir==LEFT)
            {
                char x,y;
                for(y=0;y<BLOCK_H_W;y++)
                    for(x=0;x<BLOCK_H_W;x++)
                        next_data[(BLOCK_H_W-1)-x][y]=data[y][x];
            }
            //衝突チェック
            if(can_move(point.first,point.second,next_data,map)){
                char x,y;
                for(y=0;y<BLOCK_H_W;y++)
                    for(x=0;x<BLOCK_H_W;x++)
                        data[y][x]=next_data[y][x];
                return true;
            }
        }
        return false;
    }
    //x,y 左上端座標：負になることもある
    bool can_move(int x,int y,char next_data[BLOCK_H_W][BLOCK_H_W],char map[MAP_Y][MAP_X])
    {
        char i,j;
        for(i=0;i<BLOCK_H_W;i++)
            for(j=0;j<BLOCK_H_W;j++)
                if(x+i>=0 && y+j>=0 && x+i<MAP_X && y+i<MAP_Y)
                    if(next_data[j][i]!=0 && map[y+j][x+i]!=0)
                        return false;
        return true;
    }
    bool move(Direction dir,char map[MAP_Y][MAP_X])
    {
        switch(dir){
            case RIGHT://右
                if(can_move(point.first+1,point.second,data,map)){
                    point.first+=1;
                    return true;
                }
                break;
            case LEFT://左
                if(can_move(point.first-1,point.second,data,map)){
                    point.first-=1;
                    return true;
                }
                break;
            case UP://上
                if(can_move(point.first,point.second-1,data,map)){
                    point.second-=1;
                    return true;
                }
                break;
            case DOWN://下
                if(can_move(point.first,point.second+1,data,map)){
                    point.second+=1;
                    return true;
                }
                break;
        }
        return false;
    }
}block;

void show_map(char map[MAP_Y][MAP_X])
{
    int x,y;
    for(y=0;y<MAP_Y-MAP_NOT_SHOW_NUM;y++)
        for(x=0;x<MAP_X;x++){
            attrset(COLOR_PAIR(map[y+MAP_NOT_SHOW_NUM][x]));
            mvaddstr(y,x," ");
        }
}
void show_map(block *myblock,char map[MAP_Y][MAP_X])
{
    int x,y;
    for(y=0;y<MAP_Y-MAP_NOT_SHOW_NUM;y++)
        for(x=0;x<MAP_X;x++){
            attrset(COLOR_PAIR(map[y+MAP_NOT_SHOW_NUM][x]));
            mvaddstr(y,x," ");
        }

    for(y=0;y<BLOCK_H_W;y++)
        for(x=0;x<BLOCK_H_W;x++)
            if(myblock->data[y][x]!=0 && myblock->point.second-MAP_NOT_SHOW_NUM+y>=0){
                attrset(COLOR_PAIR(myblock->data[y][x]));
                mvaddstr(myblock->point.second-MAP_NOT_SHOW_NUM+y,myblock->point.first+x," ");
            }
}

void show_nextblock(int block_num)
{
    int x,y;
    for(y=0;y<BLOCK_H_W;y++)
        for(x=0;x<BLOCK_H_W;x++)
        {
            attrset(COLOR_PAIR(block_data[block_num][y][x]));
            mvaddstr(y+NEXT_BLOCK_Y,x+NEXT_BLOCK_X," ");
        }
}

void show_score(int score)
{
    attrset(COLOR_PAIR(0));
    mvprintw(SCORE_Y,SCORE_X,"SCORE:%d",score);
}

int init()
{
    initscr();//window初期化
    noecho();//キー入力を表示しない
    cbreak();//キー入力をすぐに受け付ける
    curs_set(0);//カーソル非表示

    start_color();//カラーの初期化？
    /*色ペアの設定　(番号,前景色,背景色)
      0は前白,後黒になってるらしい*/
    init_pair(1,COLOR_BLACK,COLOR_WHITE);
    init_pair(2, COLOR_BLACK, COLOR_RED);
    init_pair(3, COLOR_BLACK, COLOR_GREEN);
    init_pair(4,COLOR_BLACK,COLOR_BLUE);
    init_pair(5,COLOR_BLACK,COLOR_YELLOW);
    init_pair(6,COLOR_BLACK,COLOR_CYAN);
    init_pair(7,COLOR_BLACK,COLOR_MAGENTA);
    init_pair(8,COLOR_BLACK,COLOR_BLUE);

    bkgd(COLOR_PAIR(0));//ターミナルの背景黒、文字白
    wtimeout(stdscr,TIMEOUT);//getchのタイムアウト設定
    keypad(stdscr, TRUE); //矢印キー使用する

    srand(time(NULL));//乱数列初期化

    return 0;
}

/*
  キー入力を反映する関数
  入力がなければfalse
*/
bool key_input(block *myblock,char map[MAP_Y][MAP_X])
{
    int keyinput;
    keyinput=getch();
    switch(keyinput){
        case KEY_RIGHT:
            myblock->move(RIGHT,map);break;
        case KEY_LEFT:
            myblock->move(LEFT,map);break;
        case KEY_UP:
            myblock->rotate(LEFT,map);break;
        case KEY_DOWN:
            myblock->move(DOWN,map);break;
        case 'c':
            endwin();
            exit(0);
        default :return false;
    }
    return true;
}

void lock_myblock(block *myblock,char map[MAP_Y][MAP_X])
{
    int x,y;
    for(y=0;y<BLOCK_H_W;y++)
        for(x=0;x<BLOCK_H_W;x++)
            if(myblock->data[y][x]!=0)map[myblock->point.second+y][myblock->point.first+x]=myblock->data[y][x];
}

/*
  揃ったラインを消す関数
  return:消したライン数
*/
int delete_lines(char map[MAP_Y][MAP_X])
{
    int x,y,cnt=0;
    for(y=MAP_Y-2;y>=0;y--){
        bool flg=true;
        for(x=1;x<MAP_X-1;x++)
            if(map[y][x]==0)flg=false;
        if(flg)cnt++;
        else if(cnt)
            for(x=1;x<MAP_X-1;x++)
                map[y+cnt][x]=map[y][x];
    }
    return cnt;
}

void gameover(char map[MAP_Y][MAP_X])
{
    int x,y;
    for(y=0;y<MAP_Y;y++)
        for(x=0;x<MAP_X;x++)
            map[y][x]=1;
    show_map(map);
}

int play(int difficulty)
{
    int next_myblock_num;
    char map[MAP_Y][MAP_X];
    block myblock;
    int score=0;
    //マップリセット
    int x,y;
    for(y=0;y<MAP_Y;y++)
        for(x=0;x<MAP_X;x++)
        {
            if(x==0 || x== MAP_X-1)map[y][x]=1;
            else if(y==MAP_Y-1)map[y][x]=1;
            else map[y][x]=0;
        }

    while(true)
    {
        for(int i=0;i<20-difficulty;i++)
        {
            key_input(&myblock,map);
            //タイム追加
            usleep(2000);
        }
        if(myblock.move(DOWN,map)==false){
            //下に落とせなかったら固定
            lock_myblock(&myblock,map);
            //上端で落とせなかったら終わり
            if(myblock.point.second<=MAP_NOT_SHOW_NUM){
                gameover(map);
                return 0;
            }
            myblock.reset(next_myblock_num);
            next_myblock_num=rand()%BLOCK_NUM;
        }
        score+=delete_lines(map)*10;
        show_map(&myblock,map);
        show_nextblock(next_myblock_num);
        show_score(score);
    }
}

int main()
{

    char nextblock;

    init();
    play(0);
    endwin();
}

~~~
