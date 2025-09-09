# 设计链表
实现 MyLinkedList 类:  
MyLinkedList() 初始化 MyLinkedList 对象。  

int get(int index)获取<mark>链表</mark>中下标为 index 的节点的值。如果下标无效，则返回 -1 。  

void addAtHead(int val) 将一个值为 val 的节点<mark>插入</mark>到链表中第一个元素之前。在插入完成后，新节点会成为链表的第一个节点。  

void addAtTail(int val) 将一个值为 val 的节点<mark>追加</mark>到链表中作为链表的最后一个元素。  

void addAtIndex(int index, int val) 将一个值为 val 的节点<mark>插入到链表中下标为 index 的节点之前</mark>。如果 index 等于链表的长度，那么该节点会被追加到链表的末尾。
如果 index 比长度更大，该节点将<mark>不会插入</mark>到链表中。  

void deleteAtIndex(int index) 如果下标有效，则<mark>删除</mark>链表中下标为 index 的节点。  

## 重点为：主要实现链表的查询、插入、删除。  
  查询其实还好，遍历就行了。但是插入要注意插入哪里？head\tail\尤其注意在这个函数里要考虑void addAtIndex(int index, int val)。删除同理  
  正因为多位置的考虑，所以为了统一逻辑，考虑设置虚拟头节点，而且可以带来更好管理链表大小信息的好处。比如初始化了MyLinkedList  
```
typedef struct {
    int size;            //记录链表大小
    Node* o_head;        //指向链表*虚拟*头节点的一个指针
} MyLinkedList;

MyLinkedList* myLinkedListCreate() {
    MyLinkedList* obj=(MyLinkedList*)malloc(sizeof(MyLinkedList));
    Node* head=(Node*)malloc(sizeof(Node));
    head->next=NULL;           //这个虚拟的头节点没有初始化val，只有一个指向真正链表第一个节点的指针
    obj->size=0;
    obj->o_head=head;          //实际上head这个Node节点与obj一起组成了一个虚拟的头节点，既方便又记录链表
    return obj;
}
```
### 查询遍历的时候不用考虑空链表等等位置元素，所以从真正的链表头节点`obj->o_head->next`开始查询，但是<mark>插入删除</mark>的时候这些因素存在影响，故一定要从虚拟头节点`obj->o_head`开始
*查询*
```
int myLinkedListGet(MyLinkedList* obj, int index) {
    if(index<0||index>=obj->size){   //每一次增加都是填满当前size下标的节点，然后size++拓宽，所以实际上size这个坐标是空的，就类似于数组下标最后一个是容量a-1，所以=size是违法不存在的
        return -1;
    }
    Node* cur=obj->o_head->next;     //初始这里会是NULL但是从初始往后全部是真正的链表节点
    while(index>0){
        cur=cur->next;
        index--;
    }
    return cur->val;
}
```
*头、尾插入*
```
void myLinkedListAddAtHead(MyLinkedList* obj, int val) {
    Node* newNode=(Node*)malloc(sizeof(Node));
    newNode->val=val;
    newNode->next=obj->o_head->next;      //当前头节点连到现在新头节点的尾巴
    obj->o_head->next=newNode;            //更新虚拟头节点后记录的真正链表头节点
    obj->size++;
}

void myLinkedListAddAtTail(MyLinkedList* obj, int val) {
    Node* newNode=(Node*)malloc(sizeof(Node));
    newNode->val=val;
    newNode->next=NULL;
    Node* cur=obj->o_head;     //万一当前链表是空的呢？所以只能令cur=虚拟头节点，(而不是o_head->next=NULL这种错误可能),遍历寻找链表末尾
    while(cur->next!=NULL){
        cur=cur->next;
    }
    cur->next=newNode;
    obj->size++;               //增删链表节点都要记得维护MyLinkedList里的size信息
}
```
*普通增加*
```
void myLinkedListAddAtIndex(MyLinkedList* obj, int index, int val) {
    Node* newNode=(Node*)malloc(sizeof(Node));
    newNode->val=val;
    newNode->next=NULL;

    if(index==obj->size){
        myLinkedListAddAtTail(obj, val);
    }
    else if(index <= 0) {
        myLinkedListAddAtHead(obj, val);
        return;
    }
    else if(index>obj->size){
        return;
    }
    
    else{
        Node* cur=obj->o_head;       //重点是等于虚拟节点而不是真实节点
        for(int i = 0; i < index; i++){
           cur = cur->next;
        }                           //我服了啊，在普通删减里把握不好index的循环条件，不如这样从前往后遍历、数直接数到index的位置
                                    //好吧我又服了，发现之前一直把握不好就是因为虚拟头节点使用不对，全部使用了真实头节点，但此处只要使用虚拟头节点，那么逻辑可统一
                                    //while(index>0){cur=cur->next;  index--;}也是正确的
        newNode->next=cur->next;
        cur->next=newNode;
        obj->size++;
    }
}
```
*普通删除以及释放内存*
```
void myLinkedListDeleteAtIndex(MyLinkedList* obj, int index) {
    if(index<0||index>=obj->size){            //前者非法，后者删无可删
        return;
    }
    else{
        Node* cur=obj->o_head;               //如上段，此处最重要的就是要等于虚拟头节点，这样涵盖了所有情况，尤其是index合法=0但是实际上这个链表它只有一个节点，那循环第一句cur就为NULL了
        for(int i = 0; i < index; i++){     
            cur = cur->next;
        }
        Node* del = cur->next;              //先把要删的节点预存起来，方便后续free
        cur->next = del->next;
        free(del);                          //一定要清空内存，但是因为前面以及删除更新为新的链表，所以不能简单写为free(cur->next)这样直接把留下来的新节点删了，会报错
        obj->size--;
    }
}

void myLinkedListFree(MyLinkedList* obj) {
    free(obj);              //虚拟的节点也要free最后只留链表
}      
```


  
