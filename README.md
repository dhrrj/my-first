# include<stdio.h>
#include<stdlib.h>
#include<time.h> 
#define D (24*60*60) 
#define H (60*60) 
#define M (60)
#define OK 1
#define ERROR 0
#define MAX_STACK_SIZE 10 /*  Stack vector size  */
typedef int StackData;
typedef int QueueData;
typedef int ElemType;
typedef struct Node
{
 int No;  /*  Car number  */
 int Timeinit; /*  Time to enter the parking lot */
}Node;
typedef struct QueueNode /*  Queue node */
{
 struct Node data; 
 struct QueueNode* next; 
} QueueNode;
typedef struct LinkQueue /*  Chain queue structure  */
{
 struct QueueNode *rear, *front;
} LinkQueue;
 
 
typedef struct SqStackNode /*  Chain stack structure  */
{ 
 int top;
 int bottom;
 struct Node stack_array[MAX_STACK_SIZE+1] ;
}SqStackNode ;
 
//***************************************************************
SqStackNode* InitStack()    /*  Initialize the stack */
{ 
 SqStackNode *S=(SqStackNode *)malloc(sizeof(SqStackNode));
 S->bottom=S->top=0; 
 return (S);
}
int FullStack(SqStackNode *S)   /*  The full stack  */
{
 return S->top==MAX_STACK_SIZE;
}
int pushStack(SqStackNode *S,Node data) /*  Into the stack  */
{ 
 if(FullStack(S))
 {
 return ERROR;  /*  Full stack, return error flag  */
 }
 S->top++ ;   
 (S->stack_array[S->top]).No=data.No ; 
 (S->stack_array[S->top]).Timeinit=data.Timeinit; 
 return OK;   /*  Pressure stack success  */
}
int popStack(SqStackNode *S,Node *data)  /* Pop top element */
{ 
 if(S->top==0)
 {
 return ERROR;  /*  Empty stack, return error flag  */
 }
 (*data).No=(S->stack_array[S->top]).No; 
 (*data).Timeinit=(S->stack_array[S->top]).Timeinit; 
 S->top--; 
 return OK; 
}
int FinfStack(SqStackNode *S,Node data) /*  Search for elements in the stack data*/
{
 int i;
 if(S->top==0)
 {
 return ERROR;  /*  Empty stack, return error flag  */
 }
 
 for(i=1;i<=S->top;i++)
 {
 if(S->stack_array[i].No == data.No)
 {
  return OK;
 }
 }
 return ERROR; 
}
 
 
 
//**************************************************** 
LinkQueue* InitQueue (void)  /*  Initialize queue  */
{
 LinkQueue *Q=( LinkQueue * ) malloc( sizeof ( LinkQueue ) );
 Q->rear=Q->front=NULL;
 return Q;
}
 int QueueEmpty ( LinkQueue *Q ) /*  An empty queue */
 {
 return Q->front == NULL;
}
 
int GetFrontQueue ( LinkQueue *Q, Node *data ) /*  Take the team first  */
{
 if ( QueueEmpty (Q) ) return 0; 
 (*data).No = (Q->front->data).Timeinit; return 1; 
}
int EnQueue ( LinkQueue **Q, Node data) /*  The team */
{
 QueueNode *p = ( QueueNode * ) malloc( sizeof ( QueueNode ) );
 (p->data).No = data.No; 
 (p->data).Timeinit = data.Timeinit; 
 p->next = NULL;
 if ( (*Q)->front == NULL ) 
 {
 (*Q)->front = (*Q)->rear = p;
 }
 else
 {
 
 (*Q)->rear = (*Q)->rear->next = p;
 }
 return 1;
}
int DeQueue ( LinkQueue **Q, Node *data) /*  Out of */
{
 if ( QueueEmpty (*Q) ) 
 {
 return 0; 
 }
 QueueNode *p = (*Q)->front; 
 (*data).No = p->data.No;   
 (*data).Timeinit = p->data.Timeinit; 
 (*Q)->front = (*Q)->front->next; 
 if ((*Q)->front == NULL) (*Q)->rear = NULL;
 free (p);
 return 1; 
}
/*********************************************************/
int now_time(void) /*  Gets the time of the day, in unit seconds */
{ 
 time_t t1; 
 time(&t1); 
 int time=t1%D; 
 return time; 
} 
 
Parking(LinkQueue **Q,SqStackNode *S) /*  parking */
{
 int i,time_now;
 Node data;
 printf("Input the Car No:\n");
 scanf(" %d",&data.No);
 
 for(i=1;i<=S->top;i++)
 {
 
 if(S->stack_array[i].No == data.No)/*  The car number already exists */
 {
 printf("The Car is existed\n");
 return ;
 }
 }
 
 EnQueue(Q,data);/*  Go in and wait in line */
 while(!QueueEmpty(*Q))
 {
 if(FullStack(S)) /*  Park stack full */
 {
 printf("Please Wait...\n");
  break;
 }
 else /*  Parking stack not full  */
 {
 DeQueue(Q,&data);/*  Wait for the train to leave  */
 data.Timeinit=now_time();/*  Record current time */
 pushStack(S,data);/*  Enter the parking stack */
 printf("Park Success\n");
 }
 }
 return ;
}
leaving(SqStackNode *S,SqStackNode *B,LinkQueue **Q)/*  leave */
{
 if(S->bottom == S->top)/*  Park stack is empty */
 {
 printf("Parking is Empty:\n");
 }
 else
 {
 Node data;
 int i,h,m,s;
 float charge; 
 int time_now,parking_time;
 printf("Leaving No:\n");
 scanf(" %d",&i);
 data.No=i;
 if(!FinfStack(S,data))/*  There is no such car in the parking lot */
 {
 printf("Do not find the car\n");
 return ;
 }
 else/*  The car is in the parking lot */
 {
 while(S->stack_array[S->top].No != i)/*  The cars behind the car in turn go out of the yield stack */
 {
 popStack(S,&data);
 pushStack(B,data);
 }
 popStack(S,&data);/*  The car is out of the parking lot */
 time_now=now_time();
 parking_time=time_now-data.Timeinit;/*  Calculate stop time */
 
 h = parking_time/H;
 parking_time = parking_time%H;
 m = parking_time/M;
 s = parking_time%M;
 charge = 6*h+0.1*(m+1);/*  Calculate parking charges */
 printf("The leaving car:%d Parking time:%d:%d:%d Charge($6/h):$%g\n",data.No,h,m,s,charge);
 
 while(B->bottom != B->top)/*  The cars in the yield stack go out of the parking stack in turn */
 {
 popStack(B,&data);
 pushStack(S,data);
 }
 while(!FullStack(S)&&(!QueueEmpty(*Q)))/*  The parking stack is not full and the waiting queue is not empty */
 {
 DeQueue(Q,&data); /*  Wait for the train to leave */
 data.Timeinit=now_time();
 pushStack(S,data);/*  The outgoing cars were parked in the parking lot */
 } 
 }
 
 }
}
situation(SqStackNode *S,LinkQueue **Q)/*  Check the current condition of the parking lot */
{
 Node data;
 int i;
 int time_now,parking_time;
 int h,m,s;
 struct QueueNode *p;
 int wait_count=0;
 p=(*Q)->front;
 if(p == NULL)/*  Waiting queue empty */
 {
 printf("Waiting car :0\n");
 }
 else/*  The wait queue is not empty */
 {
 do
 {
  wait_count++;
 p=p->next;
 }while(p!=NULL);/*  Count the number of cars in the waiting queue */
 printf("Waiting car :%d\n",wait_count);
 }
 
 printf("Car No: ");
 for(i=1;i<=S->top;i++)
 {
 printf("%-10d",S->stack_array[i].No);
 
 if(S->stack_array[i].No == data.No)
 {
  return OK;
 }
 }
 printf("\nPark time:");
 for(i=1;i<=S->top;i++)
 {
 time_now = now_time();
 parking_time = time_now - S->stack_array[i].Timeinit;/*  Calculate the current stop time */
 h = parking_time/H;
 parking_time = parking_time%H;
 m = parking_time/M;
 s = parking_time%M;
 printf("%02d:%02d:%02d ",h,m,s);
 }
 printf("\n");
 
}
 
int main()
{
 int i;
 Node data;
 SqStackNode *park;/*  Park stack */
 SqStackNode *back;/*  Make way for the stack */
 LinkQueue *wait; /*  Waiting queue */
 park=InitStack();
 back=InitStack();
 wait=InitQueue();
 while(1)
 {
 system("clear\n");
 printf("----------Welcome to our Car Parking----------\n");
 printf("  1.Parking \n");
 printf("  2.leaving \n");
 printf("  3.situation \n");
 printf("  4.exit \n");
 scanf(" %d",&i);
 switch(i)
 {
 case 1:/*  parking */
 {
 system("clear\n");
 Parking(&wait,park);
 setbuf(stdin,NULL);
 getchar();
 break;
 }
 case 2:/*  leave  */
 {
 leaving(park,back,&wait);
 setbuf(stdin,NULL);
 getchar();
 break;
 }
 case 3:/*  Check parking */
 {
 
 system("clear\n");
 situation(park,&wait);
 setbuf(stdin,NULL);
 getchar();
 break;
 }
 case 4:/*  exit */
 {
 return 0;
 }
 default:
 {
 break;
 }
 }
 }
 return 0; 
}


