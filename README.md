# D4R-algorithm---artifical-deadlock-detection
Artifical deadlock detection using MPI Send and Recive
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
#include <math.h>
#include <stddef.h>


typedef struct Node {
	int count_public;
	int count_private;
	int nodepid_public;
	int nodepid_private;
	int q_size_public;
	int q_size_private;
	int blocks;
}Node;

int main(int argc,  char **argv)
{
		// converting given block size equally
		int rank, size_mpi,deadlock=0;
		MPI_Status status;
		MPI_Init(&argc,&argv);
		MPI_Comm_rank(MPI_COMM_WORLD, &rank);
		MPI_Comm_size(MPI_COMM_WORLD, &size_mpi);
		int j=0; // queuesize
		const int nitems=7;
		int blocklength[7]={1,1,1,1,1,1,1};
		MPI_Datatype types[7]={MPI_INT,MPI_INT,MPI_INT,MPI_INT,MPI_INT,MPI_INT,MPI_INT};
		MPI_Datatype mpi_node_type;
		MPI_Aint offset[7];
		offset[0] = offsetof(Node, count_public);
		offset[1] = offsetof(Node, count_private);
		offset[2] = offsetof(Node, nodepid_public);
		offset[3] = offsetof(Node, nodepid_private);
		offset[4] = offsetof(Node, q_size_public);
		offset[5] = offsetof(Node, q_size_private);
		offset[6] = offsetof(Node, blocks);
		MPI_Type_create_struct(nitems, blocklength, offset, types, &mpi_node_type);
		MPI_Type_commit(&mpi_node_type);
		MPI_Comm_rank(MPI_COMM_WORLD, &rank);
		int i=1;
        int q[i],r[i],p[i];
        
		// initlialize all the variables in nodes
		
		   struct Node A;
				A.count_public=0;
				A.count_private=0;
				A.nodepid_public=0;
				A.nodepid_private=0;
				A.q_size_private=0;
				A.q_size_private=0;
				A.blocks=0;
		
		struct Node B;
				B.count_public=0;
				B.count_private=0;
				B.nodepid_public=1;
				B.nodepid_private=1;
				B.q_size_private=0;
				B.q_size_private=0;
				B.blocks=0;
		
		struct Node C;
				C.count_public=0;
				C.count_private=0;
				C.nodepid_public=2;
				C.nodepid_private=2;
				C.q_size_public=0;
				C.q_size_private=0;
				C.blocks=0;
	
//struct Node A;

		if(rank == 0 )
		{
		// A blocks on C using D4R algorithm by using MPi Send and Recive we are sending data from node A to node C for modification
   
			printf("\nEnter the variable in Q (array) that is linking Node A to C : ");
			scanf("%d", &q[0]);
            printf("\n A blocks C, so A initializes its D4R varaibles\n");
      		MPI_Send(&A, 1, mpi_node_type, 2, 123, MPI_COMM_WORLD);
      		MPI_Recv (&A, 1, mpi_node_type, 2, 234, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			printf("\n Modified values of A : \n count_public = %d  nodepid_public = %d q_size_public = %d \n", A.count_public,A.nodepid_public,A.q_size_public);
        	activate:	
			if(q[1] == 1)
			{
				printf("\n A activates\n");
				goto activateB;
			}


		}
		else if(rank == 2 )
		{
       	//A node data received at Node C
			MPI_Recv(&A, 1, mpi_node_type, 0, 123, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			if(A.count_public > C.count_public)
			{
				A.count_public = A.count_public + 1;
				A.count_private = A.count_public;
				A.q_size_public=1;
				A.q_size_private=1;
			}
			else
			{
				A.count_public = C.count_public + 1;
				A.count_private = A.count_public;
				A.q_size_public=1;
				A.q_size_private=1;
			}
			A.blocks=1;
			MPI_Send(&A, 1, mpi_node_type, 0, 234, MPI_COMM_WORLD);
      
        }  
      
    // B blocks on A  
	    if(rank == 1 )
        {
  
			if(p[0]!=1)
			{
				//printf("\n Node B attempts to read from P but it is empty so , B initializes its D4R varaibles and Blocks on A\n");
				MPI_Send(&B, 1, mpi_node_type, 0, 345, MPI_COMM_WORLD);
				MPI_Recv (&B, 1, mpi_node_type,0, 456, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
				printf("\n Modified values of B: \n count_public = %d nodepid_public = %d q_sizepublic = %d \n", B.count_public,B.nodepid_public,B.q_size_public);
			}
 
			else 
			{
			activateB:
				printf("\n B activates\n");
				goto activateC;
			}
  
		}
		else if(rank == 0 )
		{
			MPI_Recv(&B, 1, mpi_node_type, 1, 345, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			if(B.count_public > A.count_public)
			{
				B.count_public = B.count_public + 1;
				B.count_private = B.count_public;
				B.q_size_public=-1;
			    B.q_size_private=-1;
			}
			else
			{
				B.count_public = A.count_public + 1;
				B.count_private = B.count_public;
				B.q_size_public=-1;
				   B.q_size_private=-1;
			}
			B.blocks=1;
			MPI_Send(&B, 1, mpi_node_type, 1, 456, MPI_COMM_WORLD);
  
        }	
  // C blocks on B
  
		if(rank == 2 )
		{
			if(r[0]!=1)
			{
				//printf("\n Node C attempts to read from R but it is empty so , C initializes its D4R varaibles and Blocks on B\n");
				MPI_Send(&C, 1, mpi_node_type, 1, 567, MPI_COMM_WORLD);
				MPI_Recv(&C, 1, mpi_node_type, 1, 678, MPI_COMM_WORLD,MPI_STATUS_IGNORE);
				printf("\n Modified values of C : \n count_public = %d nodepid_public = %d q_sizepublic = %d \n", C.count_public,C.nodepid_public,C.q_size_public);
      // checking for Transmit 
			}
			else
			{
				activateC:
				printf("\n C activates\n");
				exit(0);
			}
			if(C.blocks == 1)
			{
                MPI_Send(	&C, 1, mpi_node_type, 0, 1, MPI_COMM_WORLD);
			}
      
  
		}	
		else if (rank == 1 )
		{
			MPI_Recv(&C, 1, mpi_node_type, 2, 567, MPI_COMM_WORLD, MPI_STATUS_IGNORE);
			if(C.count_public > B.count_public)
			{
				C.count_public = C.count_public + 1;
				C.count_private = C.count_public;
				C.q_size_public=-1;
				C.q_size_private=-1;
				C.blocks = 1;
			}
			else
			{
				C.count_public = B.count_public + 1;
				C.count_private = C.count_public;
				C.q_size_public=-1;
				C.q_size_private=-1;
				C.blocks = 1;
			}
			MPI_Send(&C, 1, mpi_node_type, 2, 678, MPI_COMM_WORLD);
      
		}
 // A gets Transmit from C
		else if(rank == 0)
        {
			MPI_Recv(&C, 1, mpi_node_type, 2, 1, MPI_COMM_WORLD,MPI_STATUS_IGNORE);
			if(((A.count_public < C.count_public) && (A.nodepid_public < C.nodepid_public)) || (A.count_public == C.count_public && A.nodepid_public == C.nodepid_public && A.q_size_public > abs(C.q_size_public)))
			{
				A.count_public = C.count_public;
				A.nodepid_public = C.nodepid_public;
            }
			printf("\n A gets transmit from C \n Count =%d  nodepid = %d  q_size = %d \n", A.count_public, A.nodepid_public, A.q_size_public);
            MPI_Send(&A, 1, mpi_node_type, 1, 2, MPI_COMM_WORLD);
     
        }	
		if(rank == 1)
		{
			MPI_Recv(&A, 1, mpi_node_type, 0, 2, MPI_COMM_WORLD,MPI_STATUS_IGNORE);
			if(((B.count_public < A.count_public) && (B.nodepid_public < A.nodepid_public)) || (B.count_public == A.count_public && B.nodepid_public == A.nodepid_public && abs(B.q_size_public) > A.q_size_public))
			{
				B.count_public = A.count_public;
				B.nodepid_public = A.nodepid_public;
				B.q_size_public = A.q_size_public;
			}
			printf("\n B gets transmit from A \n Count =%d  nodepid = %d  q_size = %d \n", B.count_public, B.nodepid_public, B.q_size_public);
			MPI_Send(&B, 1, mpi_node_type, 2, 3, MPI_COMM_WORLD);
		}     
		else if(rank == 2)
        {
			MPI_Recv(&B, 1, mpi_node_type, 1, 3, MPI_COMM_WORLD,MPI_STATUS_IGNORE);
			if(((C.count_public < B.count_public) && (C.nodepid_public < C.nodepid_public)) || (C.count_public == B.count_public && C.nodepid_public == B.nodepid_public && abs(C.q_size_public) >= B.q_size_public))
			{
				A.count_public = C.count_public;
				A.nodepid_public = C.nodepid_public;
				C.count_public = B.count_public;
				C.nodepid_public = B.nodepid_public;
				C.q_size_public = B.q_size_public;
			}
			printf("\n C gets transmit from B \n Count =%d  nodepid = %d  q_size = %d \n", C.count_public, C.nodepid_public, C.q_size_public);
			goto deadlock;
		}
       
		deadlock:
		if( A.count_public == C.count_public && A.nodepid_public == C.nodepid_public && A.q_size_public == C.q_size_public && A.q_size_public == A.q_size_private)
		{          
			printf("\n Deadlock has appeared\n");
			printf("\n To overcome the deadlock we increse the Q size so that the process continues\n");
			q[1]=1;
			p[0]=1;
			r[0]=1;
			goto activate;
		}
		MPI_Type_free(&mpi_node_type);	
    	MPI_Finalize();
		return 0;
}

