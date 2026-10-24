#include <stdio.h>
#include <stdlib.h>

#define MAXQ 5      
#define MAXS 10     
#define LOW 20      

struct Batch {
    int id;
    char name[30];
    int qty, price;         
    int month, year;        
    struct Batch *next;
};
struct Customer { char name[30]; int id, qty; };
struct Action { int type; struct Batch b; };     

struct Batch *head = NULL;                        
struct Customer queue[MAXQ];                     
int front = 0, rear = -1, count = 0;
struct Action stack[MAXS];                      
int top = -1;
int todayM, todayY;

int getInt(char *msg) {
    int x, r, c;
    printf("%s", msg);
    while ((r = scanf("%d", &x)) != 1) {
        if (r == EOF) exit(0);
        while ((c = getchar()) != '\n' && c != EOF);
        printf("Enter a number: ");
    }
    return x;
}

int monthsLeft(struct Batch *b) { return (b->year - todayY) * 12 + (b->month - todayM); }
int expKey(struct Batch *b)     { return b->year * 100 + b->month; }

void show(struct Batch *b) {
    int m = monthsLeft(b);
    printf("%-5d %-15s qty:%-5d Rs.%-4d exp:%02d/%d  ", b->id, b->name, b->qty, b->price, b->month, b->year);
    if (m < 0) printf("EXPIRED\n");
    else if (m <= 2) printf("EXPIRING SOON\n");
    else printf("OK\n");
}

struct Batch *findBatch(int id, int m, int y) {         
    for (struct Batch *p = head; p != NULL; p = p->next)
        if (p->id == id && p->month == m && p->year == y) return p;
    return NULL;
}

void insertSorted(struct Batch *n) {
    if (head == NULL || expKey(n) < expKey(head)) { n->next = head; head = n; return; }
    struct Batch *p = head;
    while (p->next != NULL && expKey(p->next) <= expKey(n)) p = p->next;
    n->next = p->next;
    p->next = n;
}

void removeNode(struct Batch *t) {
    if (head == t) head = t->next;
    else {
        struct Batch *p = head;
        while (p->next != t) p = p->next;
        p->next = t->next;
    }
    free(t);
}

void push(int type, struct Batch b) {
    if (top == MAXS - 1) { printf("(undo list full - this action cannot be undone)\n"); return; }
    top++;
    stack[top].type = type;
    stack[top].b = b;
}

void addBatch() {
    struct Batch *n = malloc(sizeof(struct Batch));
    if (n == NULL) { printf("Out of memory.\n"); return; }
    n->id = getInt("Medicine ID: ");
    printf("Name (one word): ");
    scanf("%29s", n->name);
    n->qty = getInt("Quantity: ");
    n->price = getInt("Price per unit (Rs): ");
    n->month = getInt("Expiry month (1-12): ");
    n->year = getInt("Expiry year: ");
    if (n->qty <= 0 || n->price <= 0 || n->month < 1 || n->month > 12 || monthsLeft(n) < 0) {
        printf("Invalid details or already expired. Not added.\n");
        free(n);
        return;
    }
    if (findBatch(n->id, n->month, n->year) != NULL) {
        printf("This batch already exists.\n");
        free(n);
        return;
    }
    insertSorted(n);
    push(1, *n);
    printf("Batch added.\n");
}

void display() {
    if (head == NULL) { printf("No stock.\n"); return; }
    for (struct Batch *p = head; p != NULL; p = p->next) show(p);
}

void deleteBatch() {
    int id = getInt("Medicine ID: ");
    int m = getInt("Expiry month: ");
    int y = getInt("Expiry year: ");
    struct Batch *p = findBatch(id, m, y);
    if (p == NULL) { printf("Batch not found.\n"); return; }
    push(2, *p);
    removeNode(p);
    printf("Batch deleted.\n");
}

void undo() {                      
    if (top < 0) { printf("Nothing to undo.\n"); return; }
    struct Action a = stack[top--];
    if (a.type == 1) {
        struct Batch *p = findBatch(a.b.id, a.b.month, a.b.year);
        if (p != NULL) removeNode(p);
        printf("Undone: addition.\n");
    } else {
        struct Batch *n = malloc(sizeof(struct Batch));
        if (n == NULL) { printf("Out of memory.\n"); return; }
        *n = a.b;
        insertSorted(n);
        printf("Undone: deletion.\n");
    }
}

void search() {
    int id = getInt("Medicine ID: "), found = 0;
    for (struct Batch *p = head; p != NULL; p = p->next)
        if (p->id == id) { show(p); found = 1; }
    if (!found) printf("Not found.\n");
}

void sortByQty() {                  
    struct Batch a[100];
    int n = 0;
    for (struct Batch *p = head; p != NULL && n < 100; p = p->next) a[n++] = *p;
    for (int i = 0; i < n - 1; i++)
        for (int j = 0; j < n - 1 - i; j++)
            if (a[j].qty > a[j + 1].qty) {
                struct Batch t = a[j]; a[j] = a[j + 1]; a[j + 1] = t;
            }
    if (n == 0) printf("No stock.\n");
    for (int i = 0; i < n; i++) show(&a[i]);
}
int sell(int id, int need) {
    if (need <= 0) { printf("Invalid quantity.\n"); return 0; }
    int avail = 0, total = 0;
    for (struct Batch *p = head; p != NULL; p = p->next)
        if (p->id == id && monthsLeft(p) >= 0) avail += p->qty;    
    if (avail < need) { printf("Cannot sell: only %d usable units available.\n", avail); return 0; }

    struct Batch *p = head;
    while (p != NULL && need > 0) {
        struct Batch *nextNode = p->next;                           
        if (p->id == id && monthsLeft(p) >= 0) {
            int take = (p->qty < need) ? p->qty : need;
            printf("Sold %d unit(s) from batch expiring %02d/%d\n", take, p->month, p->year);
            total += take * p->price;
            p->qty -= take;
            need -= take;
            if (p->qty == 0) removeNode(p);
        }
        p = nextNode;
    }
    printf("Bill: Rs. %d\n", total);
    return 1;
}

void sellMenu() {
    int id = getInt("Medicine ID: ");
    int q = getInt("Quantity: ");
    sell(id, q);
}

void joinQueue() {
    if (count == MAXQ) { printf("Queue full.\n"); return; }
    rear = (rear + 1) % MAXQ;
    printf("Customer name (one word): ");
    scanf("%29s", queue[rear].name);
    queue[rear].id = getInt("Medicine ID: ");
    queue[rear].qty = getInt("Quantity: ");
    count++;
}

void serve() {
    if (count == 0) { printf("No customers waiting.\n"); return; }
    struct Customer c = queue[front];
    front = (front + 1) % MAXQ;
    count--;
    printf("Serving %s\n", c.name);
    sell(c.id, c.qty);
}

void showQueue() {
    if (count == 0) printf("No customers waiting.\n");
    for (int i = 0; i < count; i++) {
        struct Customer *c = &queue[(front + i) % MAXQ];
        printf("%d. %s wants %d of medicine %d\n", i + 1, c->name, c->qty, c->id);
    }
}

void expiryReport() {
    int any = 0;
    for (struct Batch *p = head; p != NULL; p = p->next)
        if (monthsLeft(p) <= 2) { show(p); any = 1; }
    if (!any) printf("Nothing expired or expiring soon.\n");
}

void lowStock() {
    int any = 0;
    for (struct Batch *p = head; p != NULL; p = p->next)
        if (p->qty <= LOW && monthsLeft(p) >= 0) { show(p); any = 1; }
    if (!any) printf("No low-stock batches.\n");
}

void summary() {
    int batches = 0, units = 0, value = 0;
    for (struct Batch *p = head; p != NULL; p = p->next) {
        batches++;
        if (monthsLeft(p) >= 0) { units += p->qty; value += p->qty * p->price; }
    }
    printf("Batches: %d | Usable units: %d | Usable stock value: Rs. %d\n", batches, units, value);
}

void stockMenu() {
    printf("\n-- Manage stock --\n1 Add batch\n2 Delete batch\n3 Undo last add/delete\n0 Back\n");
    int c = getInt("Choice: ");
    if (c == 1) addBatch();
    else if (c == 2) deleteBatch();
    else if (c == 3) undo();
}

void viewMenu() {
    printf("\n-- View stock --\n1 Show all\n2 Search by ID\n3 Sort by quantity\n0 Back\n");
    int c = getInt("Choice: ");
    if (c == 1) display();
    else if (c == 2) search();
    else if (c == 3) sortByQty();
}

void queueMenu() {
    printf("\n-- Customer queue --\n1 Customer joins\n2 Serve next customer\n3 Show waiting customers\n0 Back\n");
    int c = getInt("Choice: ");
    if (c == 1) joinQueue();
    else if (c == 2) serve();
    else if (c == 3) showQueue();
}

void reportMenu() {
    printf("\n-- Reports --\n1 Expired / expiring soon\n2 Low stock\n3 Summary\n0 Back\n");
    int c = getInt("Choice: ");
    if (c == 1) expiryReport();
    else if (c == 2) lowStock();
    else if (c == 3) summary();
}

int main() {
    todayM = getInt("Enter current month (1-12): ");
    todayY = getInt("Enter current year: ");
    int ch;
    do {
        printf("\n===== PHARMACY STOCK MANAGER =====\n");
        printf("1 Manage stock (add / delete / undo)\n");
        printf("2 View stock (show / search / sort)\n");
        printf("3 Sell medicine\n");
        printf("4 Customer queue\n");
        printf("5 Reports\n");
        printf("0 Exit\n");
        ch = getInt("Choice: ");
        switch (ch) {
            case 1: stockMenu(); break;
            case 2: viewMenu(); break;
            case 3: sellMenu(); break;
            case 4: queueMenu(); break;
            case 5: reportMenu(); break;
            case 0: break;
            default: printf("Invalid choice.\n");
        }
    } while (ch != 0);
    while (head != NULL) { struct Batch *t = head; head = head->next; free(t); }
    return 0;
}
