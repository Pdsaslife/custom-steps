![image](https://github.com/user-attachments/assets/8428dcc2-2fab-4489-ac4e-8d538fc3305e)


/******************************************************************************************************

Purpose: To create a custom step that can generate SAS log and txt file as a final production batch run

By: Pritesh Desai

Date: Apr 3rd 2025

Modification: 

*..SAS 9 and SAS Viya compatible 
*..Use the code stand-alone or use it as a custom step in your viya environment 
*..Use it in EG or a flow in sas studio

********************************************************************************************************/

/* SAS templated code goes here */

%let folder_path=&folder_path;

/* Open directory and read file names */

filename sasdir "&folder_path/";

data file_list;
    length filename $256 file_path $512;
    dir_id = dopen('sasdir');

    if dir_id > 0 then do;
        do i = 1 to dnum(dir_id);
            filename = dread(dir_id, i);
            if lowcase(scan(filename, -1, '.')) = 'sas' then do;
                file_path = cats("&folder_path/", filename);
                output;
            end;
        end;
        rc = dclose(dir_id);
    end;
    drop i rc dir_id;

	run;



/* Use CALL EXECUTE to dynamically run the SAS scripts */
data _null_;
    set file_list;
    log_file = cats("&folder_path/", scan(filename, 1, '.') , ".log");
    lst_file = cats("&folder_path/", scan(filename, 1, '.') , ".txt");

    /* Submit job with log redirection */
    call execute(cats(
        '%nrstr(%let _logfile_=', log_file, ';)',
        '%nrstr(%let _lstfile_=', lst_file, ';)',
        '%nrstr(filename log_out "&_logfile_";)',
        '%nrstr(filename lst_out "&_lstfile_";)',
        '%nrstr(proc printto log=log_out print=lst_out new; run;)', 
        '%nrstr(%include "', file_path, '";)', 
        '%nrstr(proc printto; run;)'
    ));
run;








