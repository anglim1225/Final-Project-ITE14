# Final-Project-ITE14
My Project

//corrupt_queue.h

#ifndef CORRUPT_QUEUE_H
#define CORRUPT_QUEUE_H

typedef struct Client {
    char name[100];
    int isVIP; // 1 = VIP, 0 = regular
    struct Client* next;
} Client;

// Queue for regular clients
typedef struct {
    Client* front;
    Client* rear;
} RegularQueue;

// Stack for VIP clients
typedef struct {
    Client* top;
} VIPStack;

// CorruptQueue simulator
typedef struct {
    RegularQueue regularQueue;
    VIPStack vipStack;
    int supervisorPresent;
} CorruptQueue;

void initCorruptQueue(CorruptQueue* cq);
void arriveSupervisor(CorruptQueue* cq);
void leaveSupervisor(CorruptQueue* cq);
void lineupClient(CorruptQueue* cq, char* name, int isVIP);
void serveClient(CorruptQueue* cq);

#endif

// corrupt_queue.c

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include "corrupt_queue.h"

void initCorruptQueue(CorruptQueue* cq) {
    cq->regularQueue.front = cq->regularQueue.rear = NULL;
    cq->vipStack.top = NULL;
    cq->supervisorPresent = 0;
}

Client* createClient(const char* name, int isVIP) {
    Client* newClient = (Client*)malloc(sizeof(Client));
    strcpy(newClient->name, name);
    newClient->isVIP = isVIP;
    newClient->next = NULL;
    return newClient;
}

void enqueue(RegularQueue* queue, Client* client) {
    if (!queue->rear) {
        queue->front = queue->rear = client;
    } else {
        queue->rear->next = client;
        queue->rear = client;
    }
}

Client* dequeue(RegularQueue* queue) {
    if (!queue->front) return NULL;
    Client* temp = queue->front;
    queue->front = queue->front->next;
    if (!queue->front) queue->rear = NULL;
    return temp;
}

void push(VIPStack* stack, Client* client) {
    client->next = stack->top;
    stack->top = client;
}

Client* pop(VIPStack* stack) {
    if (!stack->top) return NULL;
    Client* temp = stack->top;
    stack->top = stack->top->next;
    return temp;
}

void arriveSupervisor(CorruptQueue* cq) {
    cq->supervisorPresent = 1;
    printf("Supervisor present\n");
    while (cq->vipStack.top) {
        enqueue(&cq->regularQueue, pop(&cq->vipStack));
    }
}

void leaveSupervisor(CorruptQueue* cq) {
    cq->supervisorPresent = 0;
    printf("Supervisor not here\n");
}

void lineupClient(CorruptQueue* cq, char* name, int isVIP) {
    Client* client = createClient(name, isVIP);
    if (cq->supervisorPresent || !isVIP) {
        enqueue(&cq->regularQueue, client);
        printf("%s client %s lines up at RegularQueue\n", isVIP ? "VIP" : "Regular", name);
    } else {
        push(&cq->vipStack, client);
        printf("VIP client %s lines up at VIPStack\n", name);
    }
}

void serveClient(CorruptQueue* cq) {
    Client* client = NULL;
    if (cq->vipStack.top) {
        client = pop(&cq->vipStack);
        printf("Now serving %s from VIPStack\n", client->name);
    } else if (cq->regularQueue.front) {
        client = dequeue(&cq->regularQueue);
        printf("Now serving %s from RegularQueue\n", client->name);
    } else {
        printf("No clients to serve\n");
    }
    if (client) free(client);
}

//CQSimulation.c

#include <stdio.h>
#include <string.h>
#include "corrupt_queue.h"

int main() {
    FILE* file = fopen("officeinput.txt", "r");
    if (!file) {
        printf("Could not open input file.\n");
        return 1;
    }

    CorruptQueue cq;
    initCorruptQueue(&cq);

    char line[256];
    while (fgets(line, sizeof(line), file)) {
        char command[20], name[100], type[10];
        if (sscanf(line, "lineup,%99[^,],%9s", name, type) == 2) {
            lineupClient(&cq, name, strcmp(type, "VIP") == 0);
        } else if (strncmp(line, "serve", 5) == 0) {
            serveClient(&cq);
        } else if (strncmp(line, "arrive, supervisor", 18) == 0) {
            arriveSupervisor(&cq);
        } else if (strncmp(line, "leave, supervisor", 17) == 0) {
            leaveSupervisor(&cq);
        }
    }

    fclose(file);
    return 0;
}

