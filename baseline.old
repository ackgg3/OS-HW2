#include <cstdlib>
#include <iostream>
#include <cerrno>
#include <unistd.h>
#include "mpi.h"

//run compiled code (for 5 philosophers) with mpirun -n 6 program

using namespace std;

int main ( int argc, char *argv[] ) 
{
  int id; //my MPI ID
  int p;  //total MPI processes
  MPI::Status status;
  int tag = 1;

  //  Initialize MPI.
  MPI::Init ( argc, argv );

  //  Get the number of processes.
  p = MPI::COMM_WORLD.Get_size ( );


//MY CODE, NUM FORKS
  int num_forks = p - 1;
  int total_phil = p - 1;
  int loop = 0; //my loop initiallization

  //  Determine the rank of this process.
  id = MPI::COMM_WORLD.Get_rank ( );
  
  //Safety check - need at least 2 philosophers to make sense
  if (p < 3) {
	    MPI::Finalize ( );
	    std::cerr << "Need at least 2 philosophers! Try again" << std::endl;
	    return 1; //non-normal exit
  }

bool forks[num_forks];
//initiallize array of bools for forks
for(int i = 0; i < num_forks; i++)
{
	forks[i] = true;
}

  srand(id + time(NULL)); //ensure different seeds...

//  cout << "This is process " << id << " of " << p 
//       << " I'll give you a random number: " << rand() << endl;
	
  //  Setup Fork Master (Ombudsman) and Philosophers
  if ( id == 0 ) //Master
  {
	int msgIn; //messages are integers

//MY CODE FOR MESSAGES GOING OUT FROM MASTER NODE
	int mes_Out;
	while(loop < 99)
	{  
	//let the philosophers check in
    	for (int i = 1; i < p; i++) 
	{

	     MPI::COMM_WORLD.Recv ( &msgIn, 1, MPI::INT, MPI::ANY_SOURCE, tag, status );
		

//std::cout << "Receiving message " << msgIn << " from Philosopher ";
//OLD SYNTAX		std::cout << status.getSource() << std::endl;
//std::cout << status.Get_source() << std::endl;

	  int num_phil = status.Get_source();
	  bool exit = false;

	  
	  if(num_phil != 0)
	  {
	     switch(msgIn)
	     {
		case 1://check if message recieved is a 1 (msgIn = 1)	
		{
			//Request to eat
			std::cout<<"Recieving request to eat from Philosopher "; 
			std::cout<<status.Get_source() << std::endl;		

			if(num_phil == 1) //ID is 1
			{
				//check forks, see if they are available				
				if(forks[total_phil - 1] == true && forks[0] == true)
				{
					mes_Out = 2;
					int large_fork = total_phil - 1;
					std::cout<<"Assigning forks 0 and "<<large_fork<<" to Philosopher ";
					std::cout<<status.Get_source()<<"."<<std::endl;
					forks[0] = false;
					forks[large_fork] = false;
					MPI::COMM_WORLD.Send ( &mes_Out, 1, MPI::INT,num_phil, tag );
				}
				else //go to exit
				{
					std::cout<<"Could not process Philosopher ";
					std::cout<<status.Get_source() <<"'s request, adding to ";
					std::cout<<"waiting queue."<<std::endl;
					exit = true;
				}
			}
			else //ID is any other philosopher number
			{
				if(forks[num_phil - 1] == true && forks[num_phil - 2] == true)
				{
					mes_Out = 2;
					int first_fork = num_phil - 2;
					int second_fork = num_phil - 1;
					std::cout<<"Assigning forks "<<first_fork<<" and ";
					std::cout<<second_fork<<" to Philosopher "<<status.Get_source()<<std::endl;
					forks[first_fork] = false;
					forks[second_fork] = false;
					MPI::COMM_WORLD.Send ( &mes_Out, 1, MPI::INT, num_phil, tag );
				}
				else //got to exit
				{
					std::cout<<"Could not process Philosopher ";
					std::cout<<status.Get_source() <<"'s request, adding to ";
					std::cout<<"waiting queue."<<std::endl;
					exit = true;
				}	
			}
			break;		
		}	
		case 3://if(msgIn == 3) Recieved message that you can eat
		{			
			std::cout<<"Recieving done eating from Philosopher ";
			std::cout<<num_phil<<"."<<std::endl;
			if(num_phil == 1) //put forks back
			{
				int top_fork = total_phil - 1;
				forks[top_fork] = true;
				forks[0] = true;
				exit = true;
			}
			else //put forks back
			{
				int first = num_phil - 2;
				int second = num_phil - 1;
				forks[first] = true;
				forks[second] = true;
				exit = true;
			}
			break;	
		}
	   } //end of switch case
	     //check exit bool. Should exit if forks not available, 
		 //or done eating
	   if(exit == true) //msgIn == 0, means exit function
	   {
		//exit
			std::cout<<"Philosopher "<<num_phil<<" has exited."<<std::endl;
	   }

	  }		
	}
    loop++;
   }      
  }
  else //I'm a philosopher
  {
/* NOTE: The following code sends a random integer (message) back to the server. You do 
 *
 *                            * * * * * * * * NOT * * * * * * * * 
 *
 * want to send a random message in the solution... you want a specific message (integer)
 * to mean something specific (like requesting to eat, being allowed to eat, returning 
 * resources, etc.)
 */
    //int msgOut = rand() % p; //pick a number between 0 and the number of philosophers
	int msgOut;
	int mesIn;
	

	while(true)
	{

		sleep(rand()%5); //sleep until hungry
		msgOut = 1; //set to hungry
		//check in with master - send that he would like to eat
		//send hungry request
		MPI::COMM_WORLD.Send ( &msgOut, 1, MPI::INT, 0, tag );

		MPI::COMM_WORLD.Recv ( &mesIn, 1, MPI::INT, MPI::ANY_SOURCE, tag, status );
		
		if(mesIn == 2) //Philosopher can eat
		{
			msgOut = 3; //send 'finished' message
			sleep(rand()%10); //Eat for a bit
			MPI::COMM_WORLD.Send ( &msgOut, 1, MPI::INT, 0, tag );
		}


		//receive answer to hungry request (exit or eat)

		//sending message answer (finished eating or exit)
		
		//if any other message is sent it exits
	}
	
  }
  
  //  Terminate MPI.
  MPI::Finalize ( );
  return 0;
}
