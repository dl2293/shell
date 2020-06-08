#include <sys/wait.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>
#include <string.h>
#include <iostream>
#include <queue>
#include <fcntl.h>
#include <vector>
#include <dirent.h>
#include <signal.h>
#include <errno.h>
#include <set>       //set
#include <arpa/inet.h>
#include <netdb.h>
#include <sys/socket.h>
#define STDIN 0
#define STDOUT 1
#define STDERR 2
#define FILEOUT 1
#define NPIPE 2
#define PIPE 0
#define STDERRPIPE 3
using namespace std;
vector<int> cmd_queue;
vector<int*> pipe_vector;
set <string>  Buildins;
/* 1 */
string read_cmdline()
{
	string arg;
	getline(cin, arg);
	//cerr<<arg<<endl;
	return arg;
}
void initial_path()
{
	setenv("PATH", "bin:." ,1);
}
/* 2 */
void P_env(vector<string> cmd)
{
	char* locat;
	locat = getenv(cmd[1].c_str());
	if(locat != NULL)
	{
		cerr << locat << endl;
	}
}

/* 3 */
void S_env(vector<string> cmd)
{
	setenv(cmd[1].c_str() , cmd[2].c_str(), 1);
}
/**/
void SetBuildin()
{
	char* path;
	path = getenv("PATH");
	int start_env = 0 ;
	vector<string> loc;
	string lo;
	lo = path;
	size_t found_env = lo.find(":");
	if(found_env == string::npos)
	{
		loc.push_back(lo);
	}
	else
 	{
		while(1)
		{
			string exe_cmd;
			if(found_env != string::npos)
			{
				exe_cmd = lo.substr(start_env , found_env - start_env);
				loc.push_back(exe_cmd);
				start_env = found_env + 1;
				found_env=lo.find(":", start_env);
			}
			else
			{
				exe_cmd = lo.substr(start_env, lo.length() - start_env);
				loc.push_back(exe_cmd);
				break;
			}
		}
	}
	string temp;
	struct dirent* dp;
	DIR* dirp;
	Buildins.clear();
	if(path!=NULL)
	{
		for(int i=0;i<loc.size();i++)
		{
			dirp=opendir(loc[i].c_str());
			if(dirp)
			{
				while((dp=readdir(dirp))!=NULL)
				{
					temp.assign(dp->d_name);
					if(temp.find(".")!=string::npos)
					{
						Buildins.insert(temp);
					}
				}
			}
			closedir(dirp);  
		}
	}
}

/**/
int Outpipe(vector<int*> pipe_vector, int outpipe)
{
	int pipe_in_queue = cmd_queue.size();
	int loc_in_queue = -1;//PIPE到第幾個指令在queue的哪裡
	for(int a = 0; a < pipe_in_queue; a++)
	{
		if(cmd_queue[a] == outpipe)
		{
			loc_in_queue = a;
		}
	}
	return loc_in_queue;
}

int Midoutpipe(vector<int*> pipe_vector, int g)
{
	int pipe_in_queue = cmd_queue.size();
	int loc_in_queue = -1;//PIPE到第幾個指令在queue的哪裡
	for(int a = 0; a < pipe_in_queue; a++)
	{
		//cerr<<cmd_queue[a]<<endl;
		if(cmd_queue[a] == (g + 2))
		{
			loc_in_queue = a;
		}
	}
	return loc_in_queue;
}

/**/
void test_close(int pipe_name)
{
	if(close(pipe_name) < 0)
	{
		cerr<<strerror(errno)<<endl;
	}
}
/**/
void test_dup2(int pipe_name1 , int pipe_name2)
{
	if(dup2(pipe_name1 , pipe_name2) < 0)
	{
		cerr<<strerror(errno)<<endl;
	}
}
/**/
void close_cmdpipe(int **cmd_pipe, vector<int> cmd_next_pipe,int g, int output)
{
	for(int j = 0 ; j < cmd_next_pipe.size() ; j ++)
	{
		if(cmd_next_pipe[j] != output && cmd_next_pipe[j] != g && cmd_next_pipe[j] != g-1)
		{
			close(cmd_pipe[cmd_next_pipe[j]][0]);
			close(cmd_pipe[cmd_next_pipe[j]][1]);
			//cerr<<"h\n";
		}
	}
}
void close_local_pipe(vector<int*> pipe_vector, int pipe_num, int input_pipe, int out_in_pipe)//把外面不要的pipe關了
{
	for(int j = 0 ; j < pipe_num ; j ++)
	{
		if(j != input_pipe && j != out_in_pipe)
		{
			close(pipe_vector[j][0]);
			close(pipe_vector[j][1]);
		}
	}
}
/*  */
void ChildHandler(int signo)
{
    int status;
    while(waitpid(-1,&status,WNOHANG)>0){}
}
/* */
int excfun(vector<string> cmd, vector<int> pipe_locate, int pipe_num_temp)
{
	char const *ex_line[1024];
	int k = 0 , i;
	for(i = pipe_locate[pipe_num_temp] + 1 ; i < pipe_locate[pipe_num_temp + 1] ; i ++)
	{
		ex_line[k] = cmd[i].c_str();
		k++;
	}
	ex_line[k] = NULL;
	if(execvp(ex_line[0], (char* const*)&ex_line) == -1)
	{
		if(!(Buildins.count(ex_line[0])))
		{
			cerr<<"Unknown command: ["<<ex_line[0]<< "]."<<endl;
		}
		else
		{
			cerr<<"Error(exec):"<<strerror(errno)<<endl;
		}
		exit(EXIT_FAILURE);
	}
}
/**/
int SetInput(int nowcommand)
{
	int pipe_in_queue = cmd_queue.size();
	int input_pipe = -1;
	for(int k = 0; k < pipe_in_queue; k ++)
	{
		int target = nowcommand + 1;
		if(cmd_queue[k] == target)
		{
			input_pipe = k;
		}
	}
	return input_pipe;
}
/**/
void ForkNewProcess(pid_t &pid)
{
    do{
        //殺殭屍
        signal(SIGCHLD,ChildHandler);
        //嘗試fork新的child process
        pid=fork();
        //如果失敗則sleep
        if(pid<0){
            usleep(1000);
        }
    }while(pid<0);
}
/* */
void SetInQueue(int next_loc, int cmdnum)//把NPIPE放進pipe_vector,cmdnum為總指令數
{
	int queue_cmd = cmd_queue.size();
	int temp = 0;
	for(int a = 0 ; a < queue_cmd; a++)
	{
		//cerr<<"cmdqueue: "<<cmd_queue[a]<<endl;
		if(cmd_queue[a] == (next_loc + cmdnum))
		{
			temp = 1;
		}
	}
	if(temp == 0)
	{
		int* N_pipe = new int[2];
		pipe(N_pipe);
		pipe_vector.push_back(N_pipe);
		cmd_queue.push_back(next_loc + cmdnum);
	}
}
/*  */
int findNpipe(vector<string> cmd, vector<int> pipe_locate, int g)
{
	string command = cmd[pipe_locate[g]];
	int cmdbehind = pipe_locate.size() - (g + 1); //還剩多少指令
	int nextcmd = 0;
	string NpipeNum = "";
	for(int v = 1 ; v < command.size() ; v ++)
	{
		NpipeNum += command[v];
	}
	if((command[0] == '|' || command[0] == '!') && command[1] != '\0' && command[1] != '1')//為|1以外的|N
	{
		nextcmd = stoi(NpipeNum);
		if(cmdbehind >= nextcmd)
		{
			return(g + nextcmd - 1); 
		}
		else
		{
			return (g + nextcmd + 1);
		}
		
	}
	else if((command[0] == '|' || command[0] == '!') && command[1] != '\0' && command[1] == '1')//|1的情況
	{
		if(g != (pipe_locate.size() - 1))//不是最後一個指令
		{
			return g;
		}
		else//最後一個指令
		{
			nextcmd = stoi(NpipeNum);
			return(g + nextcmd + 1);
		}
	}
	else 
	{
		return g;
	}
}
/*  */
int number_pipe(vector<string> cmd, vector<int> pipe_locate, int file)
{
	int pipe_num_temp = 0;
	queue<pid_t> child_pid;
	pid_t pid;
	//vector<vector<int>> cmd_pipe;
	int only_pipe = pipe_locate.size();
	int **cmd_pipe;
	if(file == 1)
	{
		only_pipe --;
	}
	cmd_pipe = new int*[only_pipe];
	vector<int>cmd_next_pipe;
	if(cmd[0] == "printenv")
	{
		P_env(cmd);
	}
	else if(cmd[0] == "setenv")
	{
		S_env(cmd);
	}
	else if(cmd[0] == "exit")
	{
		exit(EXIT_SUCCESS);
	}
	else
	{
		for(int g = 0; g < only_pipe; g ++)
		{
			int input_pipe = SetInput(g);
			int outpipe = findNpipe(cmd, pipe_locate, g);
			int pipe_in_queue = cmd_queue.size();
			int lastcmd = pipe_locate[g];//檢查指令是否為'|N'
			string check = cmd[lastcmd];
			int pipe_exist = 0;
			if(only_pipe > 1)
			{
				if(outpipe <= only_pipe -1)
				{
					for(int j = 0 ; j < cmd_next_pipe.size() ; j ++)
					{
						if(cmd_next_pipe[j] == g || cmd_next_pipe[j] == outpipe)
						{
							pipe_exist = 1;
						}
					}
					if(pipe_exist == 0)
					{
						if(outpipe > g)
						{
							cmd_next_pipe.push_back(outpipe);
							cmd_pipe[outpipe] = new int[2];
							pipe(cmd_pipe[outpipe]);
						}
						cmd_pipe[g] = new int[2];
						pipe(cmd_pipe[g]);
					}
				}
				else
				{
					cmd_pipe[g] = new int[2];
					pipe(cmd_pipe[g]);
				}
				
				if(g > 1)
				{
					close(cmd_pipe[g-2][0]);
					close(cmd_pipe[g-2][1]);
					delete [] cmd_pipe[g-2];
					for(int x = 0; x < cmd_next_pipe.size(); x ++)
					{
						if(cmd_next_pipe[x] == (g-2))
						{
							cmd_next_pipe.erase(cmd_next_pipe.begin() + x);
						}
					}
				}
			}
			if(g == 0)
			{
				ForkNewProcess(pid);
				if(only_pipe == 1 && check[0] != '|' && check[0] != '!')
				{
					child_pid.push(pid);
				}
				if(pid == 0)
				{
					//cerr<<"first: "<<input_pipe<<"outpipe: "<<outpipe<<endl;
					if(only_pipe > 1 && input_pipe < 0)//沒有先前的pipe而且有>一條的指令
					{
						close(STDOUT);
						if(outpipe > only_pipe -1)
						{
							int out = Outpipe(pipe_vector, outpipe);
							//cerr<<"C1_outpipe: "<<outpipe<<endl;
							close_local_pipe(pipe_vector, pipe_in_queue, -1, out);
							close(cmd_pipe[g][0]);
							close(cmd_pipe[g][1]);
							close(pipe_vector[out][0]);
							if(check[0] == '!')
							{
								close(STDERR);
								dup2(pipe_vector[out][1], STDERR);
							}
							dup2(pipe_vector[out][1], STDOUT);	
						}
						else
						{
							int midout = Midoutpipe(pipe_vector, g);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, midout); //把其他PIPE關起來
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, outpipe);
							//cerr<<"C2_outpipe: "<<cmd_queue[0]<<"Midout:"<<midout<<endl;
							if(g != outpipe)
							{
								close(cmd_pipe[g][0]);
								close(cmd_pipe[g][1]);
							}
							close(cmd_pipe[outpipe][0]);
							if(midout >= 0)
							{
								close(pipe_vector[midout][0]);
								dup2(pipe_vector[midout][1], STDOUT);
								if(check[0] == '!')
                                {
                                    close(STDERR);
                                    dup2(pipe_vector[midout][1], STDERR);
                                }
							}
							else
							{
							    if(check[0] == '!')
                                {
                                    close(STDERR);
                                    dup2(cmd_pipe[outpipe][1], STDERR);
                                }
								dup2(cmd_pipe[outpipe][1], STDOUT);
							}
						}
					}
					else if(only_pipe > 1 && input_pipe >= 0)//有先前的pipe而且有>一個的指令
					{
						if(outpipe > only_pipe -1)//|N, N>指令數
						{
							//cerr<<"C3\n";
							int out = Outpipe(pipe_vector, outpipe);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, out);
							close(STDOUT);
							close(STDIN);
							close(cmd_pipe[g][0]);
							close(cmd_pipe[g][1]);
							close(pipe_vector[input_pipe][1]);
							close(pipe_vector[out][0]);
							if(check[0] == '!')
							{
								close(STDERR);
								dup2(pipe_vector[out][1], STDERR);
							}
							dup2(pipe_vector[input_pipe][0], STDIN);
							dup2(pipe_vector[out][1], STDOUT);
						}
						else
						{
							//cerr<<"C4: "<<input_pipe<<"outpipe: "<<outpipe<<" pipe_vector:"<<pipe_vector.size();
							int midout = Midoutpipe(pipe_vector, g);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, midout); //把其他PIPE關起來
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, outpipe);
							close(STDOUT);
							close(STDIN);
							if(g != outpipe)
							{
								close(cmd_pipe[g][0]);
								close(cmd_pipe[g][1]);
							}
							close(pipe_vector[input_pipe][1]);
							close(cmd_pipe[outpipe][0]);
							
							dup2(pipe_vector[input_pipe][0], STDIN);
							if(midout >= 0)
							{
								close(pipe_vector[midout][0]);
								if(check[0] == '!')
								{
									close(STDERR);
									dup2(pipe_vector[midout][1], STDERR);
								}
								dup2(pipe_vector[midout][1], STDOUT);
							}
							else
							{
								if(check[0] == '!')
								{
									close(STDERR);
									dup2(cmd_pipe[outpipe][1], STDERR);
								}
								dup2(cmd_pipe[outpipe][1], STDOUT);
							}
						}
					}
					else if(only_pipe == 1 && input_pipe < 0)//只有一個指令&沒有之前的PIPE
					{	
						if(file == 1)//這條指令是寫檔
						{
							//cerr<<"C5_Input: "<<input_pipe<<" outpipe: "<<outpipe<<" pipe_vector: "<<pipe_vector.size()<<endl;
							close_local_pipe(pipe_vector, pipe_in_queue, -1, -1);
							int file = open(cmd[cmd.size() - 2].c_str(), O_CREAT|O_RDWR|O_TRUNC,S_IREAD|S_IWRITE);
							close(STDOUT);
							
							test_dup2(file, STDOUT);
							close(file);
						}
						else if(outpipe > only_pipe -1)
						{
							test_close(STDOUT);
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, -1);
							int out = Outpipe(pipe_vector, outpipe);
							//cerr<<"C6_Input: "<<input_pipe<<" outpipe: "<<outpipe<<" pipe_vector: "<<pipe_vector.size()<<" out: "<<out<<endl;
							close_local_pipe(pipe_vector, pipe_in_queue, -1, out);
							test_close(pipe_vector[out][0]);

							test_dup2(pipe_vector[out][1], STDOUT);
							if(check[0] == '!')
							{
								close(STDERR);
								test_dup2(pipe_vector[out][1], STDERR);
							}
						}
						else
						{
							//cerr<<outpipe<<"C7: "<<input_pipe<<" outpipe: "<<outpipe<<" pipe_vector: "<<pipe_vector.size()<<endl;
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, -1);
							close_local_pipe(pipe_vector, pipe_in_queue, -1, -1);
						}
					}
					else if(only_pipe == 1 && input_pipe >= 0) //只有一個指令&有先前PIPE
					{
						if(file == 1)//這條指令是寫檔
						{
							//cerr<<"C8\n";
							int file = open(cmd[cmd.size() - 2].c_str(), O_CREAT|O_RDWR|O_TRUNC,S_IREAD|S_IWRITE);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, -1); //把其他PIPE關起來
							close(STDOUT);
							close(STDIN);
							close(pipe_vector[input_pipe][1]);
							dup2(pipe_vector[input_pipe][0], STDIN);
							dup2(file, STDOUT);
							close(file);
						}
						else if(outpipe > g)
						{
							//cerr<<"C9\n";
							close(STDOUT);
							close(STDIN);
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, -1);
							int out = Outpipe(pipe_vector, outpipe);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, out);
							close(pipe_vector[input_pipe][1]);
							close(pipe_vector[out][0]);
							if(check[0] == '!')
							{
								close(STDERR);
								dup2(pipe_vector[out][1], STDERR);
							}
							dup2(pipe_vector[input_pipe][0], STDIN);
							dup2(pipe_vector[out][1], STDOUT);
						}
						else
						{
							//cerr<<"C10_input: "<<input_pipe<<"outpipe: "<<outpipe<<"cmd_queue:"<<cmd_queue.size()<<endl;
							close(STDIN);
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, -1);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, -1);
							close(pipe_vector[input_pipe][1]);
							
							dup2(pipe_vector[input_pipe][0], STDIN);
						}
					}
					char const *ex_line[1024];
					int k = 0 , i;
					for(i = 0 ; i < pipe_locate[0] ; i ++)
					{
						ex_line[k] = cmd[i].c_str();
						// cerr<<ex_line[k]<<endl;
						k++;
					}
					ex_line[k] = NULL;
					if(execvp(ex_line[0], (char* const*)&ex_line) == -1)
					{
						if(!(Buildins.count(ex_line[0])))
						{
							cerr<<"Unknown command: ["<<ex_line[0]<< "]."<<endl;
						}
						else
						{
							cerr<<"Error(exec):"<<strerror(errno)<<endl;
						}
						exit(EXIT_FAILURE);
					}
				}
				
			}
			else
			{
				ForkNewProcess(pid);
				if(g == only_pipe - 1 && check[0] != '|' && check[0] != '!')
				{
					child_pid.push(pid);
				}
				if (pid == 0)
				{
					if(g == only_pipe - 1 && check[0] != '|' && check[0] != '!')//最後一個指令不是'|N' or '!N'
					{
						if(file == 1)
						{
							//cerr<<"C11\n";
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, -1);
							int file = open(cmd[cmd.size() - 2].c_str(), O_CREAT|O_RDWR|O_TRUNC,S_IREAD|S_IWRITE);
							close(STDIN);
							close(STDOUT);
							close(cmd_pipe[g-1][1]);
							close(cmd_pipe[g][0]);
							close(cmd_pipe[g][1]);
							
							if(input_pipe >= 0)
							{
								close(pipe_vector[input_pipe][1]);
								close(cmd_pipe[g-1][0]);
								dup2(pipe_vector[input_pipe][0] , STDIN);
							}
							else
							{
								dup2(cmd_pipe[g-1][0] , STDIN);
							}
							dup2(file, STDOUT);
							close(file);
						}
						else
						{
							//cerr<<"C12_INput: "<<input_pipe<<" outpipe: "<<outpipe<<" cmd_queue_size: "<<cmd_queue.size()<<endl;
							close(STDIN);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, -1);
							close(cmd_pipe[g][0]);
							close(cmd_pipe[g][1]);
							close(cmd_pipe[g-1][1]);
							if(input_pipe >= 0)
							{
								close(pipe_vector[input_pipe][1]);
								close(cmd_pipe[g-1][0]);
								dup2(pipe_vector[input_pipe][0] , STDIN);
							}
							else
							{
								dup2(cmd_pipe[g-1][0] , STDIN);
							}
						}
					}
					else if(g == only_pipe - 1 && check[0] == '|')//最後一個指令是'|N'
					{
						close(STDOUT);
						close(STDIN);
						int out = Outpipe(pipe_vector, outpipe);
						//cerr<<"C13_INput: "<<input_pipe<<" outpipe: "<<outpipe<<" cmd_queue_size: "<<cmd_queue.size()<<" out: "<<out<<endl;
						close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, out);
						close(cmd_pipe[g-1][1]);
						test_close(cmd_pipe[g][0]);
						test_close(cmd_pipe[g][1]);
						close(pipe_vector[out][0]);
						if(input_pipe >= 0)
						{
							close(pipe_vector[input_pipe][1]);
							close(cmd_pipe[g-1][0]);
							dup2(pipe_vector[input_pipe][0] , STDIN);
						}
						else
						{
							dup2(cmd_pipe[g-1][0] , STDIN);
						}
						dup2(pipe_vector[out][1], STDOUT);
					}
					else if(g == only_pipe - 1 && check[0] == '!') //最後一個指令是'!N'
					{
						close(STDOUT);
						close(STDIN);
						close(STDERR);
						int out = Outpipe(pipe_vector, outpipe);
						//cerr<<"C14_INput: "<<input_pipe<<" outpipe: "<<outpipe<<" cmd_queue_size: "<<cmd_queue.size()<<" out: "<<out<<endl;
						close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, out);
						close(cmd_pipe[g-1][1]);
						close(pipe_vector[out][0]);
						close(cmd_pipe[g][0]);
						close(cmd_pipe[g][1]);
						if(input_pipe >= 0)
						{
							close(pipe_vector[input_pipe][1]);
							close(cmd_pipe[g-1][0]);
							dup2(pipe_vector[input_pipe][0] , STDIN);
						}
						else
						{
							dup2(cmd_pipe[g-1][0] , STDIN);
						}
						dup2(pipe_vector[out][1], STDERR);
						dup2(pipe_vector[out][1], STDOUT);
					}
					else //中間指令
					{
						if(outpipe > only_pipe -1)
						{
							//cerr<<"C15\n";
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, -1);
							int out = Outpipe(pipe_vector, outpipe);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, out);
							close(STDOUT);
							close(STDIN);
							close(cmd_pipe[g][0]);
							close(cmd_pipe[g][1]);
							close(cmd_pipe[g-1][1]);
							close(pipe_vector[out][0]);
							if(check[0] == '!')
							{
								close(STDERR);
								dup2(pipe_vector[out][1], STDERR);
							}
							if(input_pipe >= 0)
							{
								close(pipe_vector[input_pipe][1]);
								close(cmd_pipe[g-1][0]);
								dup2(pipe_vector[input_pipe][0] , STDIN);
							}
							else
							{
								dup2(cmd_pipe[g-1][0] , STDIN);
							}
							dup2(pipe_vector[out][1], STDOUT);
						}
						else
						{
							//cerr<<"C16_outpipe:"<<outpipe<<endl;
							int midout = Midoutpipe(pipe_vector, g);
							close_local_pipe(pipe_vector, pipe_in_queue, input_pipe, midout); //把其他PIPE關起來
							close_cmdpipe(cmd_pipe, cmd_next_pipe, g, outpipe);
							close(STDOUT);
							close(STDIN);
							close(cmd_pipe[g-1][1]);
							close(cmd_pipe[outpipe][0]);

							if(input_pipe >= 0)
							{
								close(pipe_vector[input_pipe][1]);
								close(cmd_pipe[g-1][0]);
								dup2(pipe_vector[input_pipe][0] , STDIN);
							}
							else
							{
								dup2(cmd_pipe[g-1][0] , STDIN);
							}
							if(midout >= 0)
							{
								if(check[0] == '!')
								{
									close(STDERR);
									dup2(pipe_vector[midout][1], STDERR);
								}
								close(pipe_vector[midout][0]);
								dup2(pipe_vector[midout][1], STDOUT);
							}
							else
							{
								if(check[0] == '!')
								{
									close(STDERR);
									dup2(pipe_vector[outpipe][1], STDERR);
								}
								dup2(cmd_pipe[outpipe][1], STDOUT);
							}
						}
					}
					excfun(cmd, pipe_locate, pipe_num_temp);
				}
				pipe_num_temp ++;
			}
		}//for迴圈尾
	}
	if(only_pipe >= 2)
	{
		test_close(cmd_pipe[only_pipe-1][0]);
		test_close(cmd_pipe[only_pipe-1][1]);
		test_close(cmd_pipe[only_pipe-2][0]);
		test_close(cmd_pipe[only_pipe-2][1]);
		delete[] cmd_pipe[only_pipe-1];
		delete[] cmd_pipe[only_pipe-2];
	}
	
	for(int y = 0; y < cmd_queue.size(); y ++)
	{
		cmd_queue[y] -= only_pipe;
	}
	for(int r = 0 ; r < cmd_queue.size(); r++)
	{
		if(cmd_queue[r] <= 0)
		{
			close(pipe_vector[r][0]);
			close(pipe_vector[r][1]);
			delete[] pipe_vector[r];
			pipe_vector.erase(pipe_vector.begin() + r);
			cmd_queue.erase(cmd_queue.begin() + r);
		}
	}
	//cerr<<"cmd_queue:"<<cmd_queue.size()<<endl;
	while(!(child_pid.empty()))
	{
		int status;
		waitpid(child_pid.front(),&status,0);
		child_pid.pop();
	}
}
/*  */
int exe_cmd(vector<string> cmd, vector<int> pipe_locate)
{
	//cerr<<"exe\n";
	int fi = 0;
	int only_pipe = pipe_locate.size();
	if(only_pipe != 1)
	{
		if(cmd[pipe_locate[only_pipe - 2]] == ">")
		{
			fi = 1;
		}
	}
	number_pipe(cmd, pipe_locate, fi);
	return 1;
}
/*  */
int cmp_cmdline(string arg) // separate string
{
	vector<int> pipe_locate;
    int start = 0, have_cmd = 0;
	vector<string> cmd; //command in line
    size_t found = arg.find(" ");
    if(found == string::npos)
	{
		cmd.push_back(arg);
	}
	else
 	{
		while(1)
		{
			string cmd_content;
			if(found != string::npos)
			{
				cmd_content = arg.substr(start , found - start);
				cmd.push_back(cmd_content);
				start = found + 1;
				found=arg.find(" ", start);
				if(cmd.back() == "")
				{
					cmd.pop_back();
				}
				else if(cmd.back() == "|" || cmd.back() == ">" || cmd.back() == "!")
				{
					pipe_locate.push_back(cmd.size() - 1);
				}
				else if((cmd[cmd.size()-1][0] == '|' || cmd[cmd.size()-1][0] == '!') && cmd[cmd.size()-1][1] != '\0')
				{
					pipe_locate.push_back(cmd.size() - 1);
				}
			}
			else
			{
				cmd_content = arg.substr(start, arg.length() - start);
				cmd.push_back(cmd_content);
				if(cmd.back() == "")
				{
					cmd.pop_back();
				}
				else if(cmd.back() == "|" || cmd.back() == ">" || cmd.back() == "!")
				{
					pipe_locate.push_back(cmd.size() - 1);
				}
				else if((cmd[cmd.size()-1][0] == '|' || cmd[cmd.size()-1][0] == '!') && cmd[cmd.size()-1][1] != '\0')
				{
					pipe_locate.push_back(cmd.size() - 1);
				}
				break;
			}
		}
	}
	string lastcmd = cmd.back();
	int cmdnum = pipe_locate.size();
	if(lastcmd[0] != '|' && lastcmd[0] != '!')
	{
		cmdnum ++;
	}
	for(int i = 0; i < pipe_locate.size(); i ++)
	{
		string checkNpipe = cmd[pipe_locate[i]];
		string NpipeNum = "";
		for(int v = 1 ; v < checkNpipe.size() ; v ++)
		{
			NpipeNum += checkNpipe[v];
		}
		if((checkNpipe[0] == '|' || checkNpipe[0] == '!') && checkNpipe[1] != '\0')
		{
			int next_loc = 0;
			int numpipe = stoi(NpipeNum);
			if(lastcmd[0] != '|' && lastcmd[0] != '!')
			{
				next_loc =  numpipe - (pipe_locate.size() - i);
			}
			else if(lastcmd[0] == '|' || lastcmd[0] == '!')
			{
				next_loc = numpipe - (pipe_locate.size() - i - 1);
			}
			if(next_loc > 0)
			{
				SetInQueue(next_loc, cmdnum);
			}
		}
	}
	if(lastcmd[0] != '|' && lastcmd[0] != '!')
	{
		pipe_locate.push_back(cmd.size()); //結尾位子
		cmd.push_back(to_string(cmd.size()));
	}
	if(cmd[0]!= "")
	{
		exe_cmd(cmd, pipe_locate);
	}
	return 1;
}

int main(int argc, char **argv)
{
    initial_path();
	int state = 1;
	SetBuildin();
	string arg;
	do
	{
		cout << "% ";
		arg = read_cmdline();
		state = cmp_cmdline(arg);
	}while(state);
	return 0;
}
