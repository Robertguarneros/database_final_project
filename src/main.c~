#include <stdio.h>
#include <stdbool.h>
#include <getopt.h>
#include <stdlib.h>

#include "common.h"
#include "file.h"
#include "parse.h"

void print_usage(char *argv[]) {
	printf("Usage: %s -n -f <database file>\n", argv[0]);
	printf("\t -n - create a new database file\n");
	printf("\t -f - (required) path to database file\n");
	printf("\t -a - add a new employee. Example format: \"EmployeeName,Address,HoursWorked\"\n");
	printf("\t -l - List the employees in the file\n");
	printf("\t -r - Remove employee <employee name>\n");
	printf("\t -u - Update employee hours. Example format:\"EmployeeName,Hours\"\n");

	return;
}

int main(int argc, char *argv[]){
	char *filepath = NULL;
	char *addstring = NULL;
	bool newfile = false;
	bool list = false;
	bool remove = false;
	bool update = false;
	char *employee_name = NULL;
	char *update_information = NULL;
	int c;
	
	int dbfd = -1;
	struct dbheader_t *dbhdr = NULL;
	struct employee_t *employees = NULL;
	while ((c = getopt(argc, argv, "nf:a:lr:u:")) != -1) {
		switch (c) {
			case 'n':
				newfile = true;
				break;
			case 'f':
				filepath = optarg;
				break;
			case 'a':
				addstring = optarg;
				break;
			case 'l': 
				list = true;
				break;
			case 'r':
				remove = true;
				employee_name = optarg;
				break;
			case 'u':
				update = true;
				update_information = optarg;
				break;
			case '?':
				printf("Unkown option -%c\n",c);
				break;
			default:
				return -1;
		}
	}
	if (filepath == NULL) {
		printf("Filepath is a required argument\n");
		print_usage(argv);
	}		
	if (newfile) {
		dbfd = create_db_file(filepath);
		if (dbfd == STATUS_ERROR) {
			printf("Unable to create database file\n");
			return -1;
		}
		if (create_db_header(dbfd, &dbhdr) == STATUS_ERROR) {
			printf("Failed to create database header\n");
			return -1;
		}
	} else {
		dbfd = open_db_file(filepath);
		if (dbfd == STATUS_ERROR) {
			printf("Unable to open database file\n");
			return -1;
		}
		if (validate_db_header(dbfd, &dbhdr) == STATUS_ERROR) {
			printf("Failed to validate database header\n");
			return -1;	
		}
	}
	

	if(read_employees(dbfd,dbhdr, &employees) != STATUS_SUCCESS) {
		printf("Failed to read employees");
		return -1;
	}
	if(addstring) {
		dbhdr->count++;
		employees = realloc(employees,dbhdr->count*(sizeof(struct employee_t)));
		add_employee(dbhdr, employees, addstring);
	}	
	if(remove) {
		if(remove_employee(dbhdr,employees, employee_name) != STATUS_SUCCESS) {
			printf("Error removing employee %s. Employee does not exist.\n", employee_name);
			return -1;
		}
	}
	if(update) {
		if(update_employee_hours(dbhdr, employees, update_information) != STATUS_SUCCESS) {
			printf("Error updating hours. Employee does not exist.\n");
			return -1;
		}

	}
	if(list) {
		list_employees(dbhdr,employees);
	}
	if (output_file(dbfd,dbhdr,employees) != STATUS_SUCCESS) {
		printf("Error when writing file");
		return -1;
	}
	//printf("Newfile: %d\n", newfile);
	//printf("Filepath: %s\n",filepath);
	return 0;
}
