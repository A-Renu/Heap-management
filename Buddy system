#include<stdio.h>
#include<stdlib.h>
#include<math.h>

#define HEAP_SIZE 1024*16
typedef struct freeList freeList;
typedef struct allottedList allottedList;
typedef struct Node Node;
typedef enum {NOT_UPDATED, UPDATED} status;
char HEAP[HEAP_SIZE];

struct freeList //DLL of free spaces
{
    int index;
    int offset;
    freeList* next;
    freeList* prev;
};

struct allottedList //SLL of allocated spaces
{
    int index;
    int offset;
    char* ptr;
    allottedList* next;
};

struct Node //Node containing pointers of both the lists
{
    freeList* head; //pointer to freeList
    allottedList* start; //pointer to allocatedList
    status label; //status of freeList
};

Node* list_ptr; //pointer to the Node

//Prototypes
void initialise();
char* newAlloc(int size);
int powerOfTwo(int size);
char* allocate(int size);
void split(freeList* nptr);
char* modifyList(freeList* nptr);
void add(freeList* nptr, char* ans);
void free_(char* lptr);
void add_node(freeList* ptr);
void updateList(freeList* lptr);
void print_freeList(freeList* head);
void print_allottedList(allottedList* head);
//END

void initialise() //initialises the node and freeList
{
    freeList* nptr;
    nptr = (freeList*)malloc(sizeof(freeList));
    nptr->index = 0;
    nptr->offset = HEAP_SIZE;
    nptr->next = NULL;
    nptr->prev = NULL;
    list_ptr->label = UPDATED;
    list_ptr->head = nptr;
    list_ptr->start = NULL;
}

char* newAlloc(int size) //rounds off to the nearest power of 2
{
    char* ans = NULL;
    int pos;
    pos = ceil(log2(size));
    size =  pow(2,pos);
    ans = allocate(size);
    return ans;
}

char* allocate(int size) //allocated spaces for a request
{
    freeList *ptr, *head, *lptr;
    char* ans = NULL;
    int max = HEAP_SIZE+1;
    int flag = 1;
    head = list_ptr->head;
    lptr = head;
    while(head && flag)
    {
        if(head->offset > size && head->offset <= max)
        {
            max = head->offset;
            ptr = head; //node with just greater value
        }
        else if(head->offset == size)
        {
            ans = modifyList(head);
            list_ptr->label = NOT_UPDATED;
            flag = 0;
        }
        if(head->next == NULL)
        {
            lptr = head; //last node
        }
        head = head->next;
    }
    if(list_ptr->head == NULL)
    {
    	  list_ptr->label = UPDATED;
    }
    if(max == HEAP_SIZE+1 && flag)
    {
        if(list_ptr->label) //updated list
        {
            printf("------------------------------------\n");
            printf("| FAILURE                           |\n");
            printf("| Can't allocate space of size %d  |\n",size);
            printf("| Not Enough Space                  |\n");
            printf("------------------------------------\n");
        }
        else if(lptr)//list not yet updated
        {
            updateList(lptr);
            allocate(size);
        }
    }
    else if(flag == 1) //no space of equal size
    {
L:
        split(ptr);
        ptr = ptr->next;
        if(ptr->offset == size)
        {
            ans = modifyList(ptr);
            list_ptr->label = NOT_UPDATED;
        }
        else
        {
            goto L;
        }
    }
    return ans;
}

void split(freeList* nptr) //splits a chunk of memory into two equal parts
{
    freeList *lptr;
    lptr = (freeList*)malloc(sizeof(freeList));
    lptr->next = nptr->next;
    lptr->prev = nptr;
    if(nptr->next)
    {
        (lptr->next)->prev = lptr;
    }
    nptr->next = lptr;
    nptr->offset/=2;
    lptr->offset = nptr->offset;
    lptr->index = (nptr->index)+(nptr->offset);
}

char* modifyList(freeList* nptr) //removes the allocated memory from freeList
{
    char *ans;
    freeList *last, *frwd;
    last = nptr->prev;
    frwd = nptr->next;
    if(last)
    {
        last->next = nptr->next;
    }
    else
    {
        list_ptr->head = nptr->next;
    }
    if(frwd)
    {
        frwd->prev = last ;
    }
    ans = &HEAP[nptr->index]; //pointer to the memory allocated
    add(nptr, ans);
    free(nptr);
    return ans;
}

void add(freeList* nptr, char* ans) //adds the allocated space to the allocatedList
{
    allottedList *lptr, *ptr;
    lptr = (allottedList*)malloc(sizeof(allottedList));
    lptr->index = nptr->index;
    lptr->offset = nptr->offset;
    lptr->ptr = ans;
    lptr->next = NULL;
    ptr = list_ptr->start;
    if(ptr)
    {
        lptr->next = ptr;
    }
    list_ptr->start = lptr; //adding at the start
}

void free_(char* lptr) //deletes the node from allocatedList
{
    allottedList *curr, *last;
    freeList* node;
    last = list_ptr->start;
    if(lptr)
    {
        if(last->ptr == lptr)
        {
            curr = last;
            list_ptr->start = curr->next;
        }
        else
        {
            curr = last->next;
            while(curr->ptr != lptr)
            {
                curr = curr->next;
                last = last->next;
            }
            last->next = curr->next;
        }
        node = (freeList*)malloc(sizeof(freeList));
        node->index = curr->index;
        node->offset = curr->offset;
        free(curr);
        add_node(node);
    }
}

void add_node(freeList* ptr)//adds the free node to the freeList
{
    freeList* lptr;
    lptr = list_ptr->head;
    if(lptr == NULL)
    {
        list_ptr->head = ptr;
        ptr->next = ptr->prev = NULL;
    }
    else
    {
        while(lptr->index < ptr->index && lptr->next != NULL)
        {
            lptr = lptr->next;
        }
        if(lptr->next == NULL && lptr->index < ptr->index)
        {
            lptr->next = ptr;
            ptr->prev = lptr;
            ptr->next = NULL;
        }
        else
        {
            if(lptr->prev)
            {
                (lptr->prev)->next = ptr;
            }
            else
            {
                list_ptr->head = ptr;
            }
            ptr->prev = lptr->prev;
            ptr->next = lptr;
            lptr->prev = ptr;
        }
    }
}

void updateList(freeList* lptr) //merges the buddies
{
	if(lptr)
	{
    freeList *before, *after;
    int index;
    int flag = 0;
    before = lptr->prev;
    after = lptr->next;
    if(before)
    {
        index = (before->index)+(before->offset);
        if(index == lptr->index && before->offset == lptr->offset)
        {
            before->offset*=2;
            before->next = lptr->next;
            if(after)
            {
                after->prev = before;
            }
            free(lptr);
            lptr = before;
            flag = 3;
        }
    }
    if(after)
    {
        index = (lptr->index)+(lptr->offset);
        if(index == after->index && lptr->offset == after->offset)
        {
            lptr->offset*=2;
            lptr->next = after->next;
            if(after->next)
            {
                (after->next)->prev = lptr;
            }
            free(after);
            flag++;
        }
    }
    if(flag == 0)//no merge
    {
        lptr = lptr->prev;
    }
    else if(flag == 3)//merged with only before node
    {
        lptr = before;
    }
    if(lptr)
    {
        updateList(lptr);
    }
   }
    list_ptr->label = UPDATED;
}

void print_freeList(freeList* head)//prints the freeList
{
    freeList* ptr = NULL;
    if(head)
    {
        printf("Free List :- \n\n");
        while(head)
        {
            printf("Index\t : %d\n",head->index);
            printf("Offset\t : %d\n",head->offset);
            printf("\n");
            head = head->next;
        }
    }
    else
    {
        printf("!! Empty Free List, No free space !!");
    }
    printf("\n");
}

void print_allottedList(allottedList* head)//prints the allocated List
{
    if(head)
    {
        printf("------------------------------------\n");
        printf("| Allocated List :- \n");
        printf("|\n");
        while(head)
        {
            printf("|  Index\t : %d\n",head->index);
            printf("|  Offset\t : %d\n",head->offset);
            printf("|\n");
            head = head->next;
        }
    }
    else
    {
        printf("!! Empty Allocated List, No allocations !!\n");
    }
    printf("\n");
}

int main()
{
    list_ptr = (Node*)malloc(sizeof(Node));
    initialise();
    
    char* ptr[5] = {NULL};
    
    ptr[1] = newAlloc(3);
    free_(ptr[1]);
    ptr[2] = newAlloc(4);
    ptr[3] = newAlloc(4);
    ptr[4] = newAlloc(500);
    free_(ptr[2]);
    free_(ptr[3]);
    free_(ptr[4]);
    ptr[5] = newAlloc(1000);
    ptr[1] = newAlloc(512);
    print_freeList(list_ptr->head);
    print_allottedList(list_ptr->start);

    return 0;
}
