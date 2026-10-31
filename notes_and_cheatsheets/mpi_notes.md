# MPI - Message Passing Interface
#### include <mpi.h>

An interface that allows multiple processes to communicate with each other while while executing a prallel program.

Since they are processes, memories are separate unlike threads where memory is shared(concurrency).

## Main terms

### Process:
an independently executing instance of the program
### Communicator:
defines the group and communication context
### Rank:
a numerical identifier of a process withing a communicator
### Message:
data sent from one process to another
### Buffer: 
memory used to hold outgoing or incoming data

MPI - is a specification, not an implementation

## Explained

### Blocking point-to-point communication

a communication operation connects one sender with one receiver.
E.g: 
- Process 0 sends the number 42.
- Process 1 receives the number 42.
- Other processes do not participate in this particular transfer.

#### Blocking:

==> means that the calling thread does not continue past the MPI operation until that operation satisfies its completion condition.

- MPI_Send completes when its send buffer can safely be reused.
- MPI_Recv completes when the matching message has been received into the destination buffer.

Note: a completed MPI_Send =! receiver obtained message 4rm MPI_Recv (not necessarily)

##### How does MPI know which message belongs to which receiver?

```
Message:
    Source       = 0
    Destination  = 1
    Tag          = 0
    Communicator = MPI_COMM_WORLD
    Payload      = 42
```

source and tag can be wildcaded: MPI_ANY_SOURCE n MPI_ANY_TAG

There are 2 approaches that can happen in the systems:
* Eager protocol:
often with small messages where data can be buffered and send can complete. Here the mpi buffered it even before the rank posts its receive.

* Rendezvous protocol
often for larger messages, sender requests to transfer and receiver replies with ready then data is transfered. This depends on the mpi library, networkhardware ...

##### Deadlocks ?

- Possible where rank 0 MPI_Send is waiting for rank 1's receive while Rank 1 MPI_Send is waiting for rank 0's receive.

##### Solution
- MPI_Sendrecv:
coordinates a send and receive within one operation
- carefully ordered non-blocking communication

##### related functions:
MPI_Ssend - buffered send - A synchronous send gives stronger assurance of receiver participation - does not mean the receiver has necessarily returned from MPI_Recv yet.

MPI_Bsend - buffered send - The programmer attaches buffer storage using MPI_Buffer_attach - MPI copies the outgoing data into that storage so the application can reuse its original buffer after the buffered send completes.- The actual delivery to the receiver may still be pending. The attached buffer must have sufficient space.

MPI_Rsend - ready send - The sender promises that the matching receive has already been started. - If the receive has not been posted, the program violates the operation's precondition and behavior is erroneous. - It is not automatically faster than standard sending.

### Non-blocking communication
initiates sending/receiving and returns without waiting for the overall transfer to finish.

- MPI_Isend()
- MPI_Irecv()
- MPI_Wait()
- MPI_Test()

Non-blocking separate 2 moments

- Starting an operation.
- Completing that operation

Where letter "I" means "immediate"

E.g:
```
int y;

MPI_Irecv(&y, 1, MPI_INT, 0, 0, comm, &req);

printf("%d\n", y); // receive not completed, non blocking

MPI_Wait(&req, MPI_STATUS_IGNORE);
```

##### MPI_Wait versus MPI_Test

- ##### MPI_Wait:
    blocks until the request completes.
- ##### MPI_Test:
    checks completion and returns immediately.


### Persistent communication

useful when the same communication pattern is repeated many times.

Initialization establishes a reusable communication description:
- Buffer address
- Datatype
- Element count
- Destination or source
- Tag
- Communicator
MPI_Start activates the request.

MPI_Wait completes that activation, returning the persistent request to its inactive state.

You cannot safely modify its active buffer in conflict with MPI, and you cannot restart an already active persistent request.

Key distinction: MPI_Send_init prepares an operation. It does not actually send the message.

### Derived datatypes and memory layout

- MPI_Type_vector

- MPI_Type_commit

- MPI_Pack

- MPI_Unpack

By default, MPI can communicate ordinary contiguous data.

Eg:

```
int numbers[4] = {10, 20, 30, 40};
MPI_Send(numbers, 4, MPI_INT, 1, 0, comm);
```

The four integers occupy consecutive array positions.
But sometimes we want to send non-contiguous memory.

E.g: transferring one column of a matrix

A derived datatype lets us describe this pattern without manually copying the column into a temporary contiguous array.

```
MPI_Datatype column_type;

MPI_Type_vector(
    4,          // count: four blocks
    1,          // one element per block
    4,          // distance between block starts
    MPI_INT,
    &column_type
);

MPI_Type_commit(&column_type);

MPI_Send(&matrix[0][1], 1, column_type,
         1, 0, comm);

MPI_Type_free(&column_type);
```

#### MPI_Pack and MPI_Unpack
These functions explicitly serialize and deserialize data into/from a packed message buffer.

### Collective communication