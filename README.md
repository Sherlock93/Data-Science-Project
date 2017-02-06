# Data-Science-Project
Data Incubator Challenge question project;  
Project Name: Probability of Default Model;  
Software Used: SAS



                                                                                                                                        
                                                                                                                                        
/*Importing data*/                                                                                                                      
libname in "E:\SAS\Biswarup\Project";                                                                                    
                                                                                                                                        
/*To check the overall structure of the data*/                                                                                          
proc contents data = in.Data_logistic;                                                                                                  
run;                                                                                                                                    
proc print;run;                                                                                                                         
/*----------------------------------------------------------------------------------------------------------*/                          
                                                                                                                                        
/*To see the percentage distribution of the target variable*/                                                                           
proc freq data = in.Data_logistic;                                                                                                      
table Is_Good * Income_Group;                                                                                                           
run;                                                                                                                                    
                                                                                                                                        
                                                                                                                                        
/*To check if this is a good target variable or not and whether 0 is good or 1*/                                                        
proc freq data = in.Data_Logistic;                                                                                                      
table Is_Good * Income_Group/ norow nocol;                                                                                              
run;                                                                                                                                    
                                                                                                                                        
proc freq data = in.Data_Logistic;                                                                                                      
table Is_Good * Residential_Status/ norow nocol;                                                                                        
run;                                                                                                                                    
                                                                                                                                        
/*Since the percentage of good should usually be more, we take 0as good 1 as bad*/                                                      
/*----------------------------------------------------------------------------------------------------------*/                          
                                                                                                                                        
/*To create a macro to check proc freq of the categorical variables*/                                                                   
proc freq data = in.Data_Logistic ;                                                                                                     
table Is_Good/ norow nocol ;                                                                                                            
run;                                                                                                                                    
                                                                                                                                        
%macro Pfreq(Var);                                                                                                                      
   proc freq data = in.Data_Logistic ;                                                                                                  
   table &var./ norow nocol ;                                                                                                           
   run;                                                                                                                                 
%mend Pfreq;                                                                                                                            
                                                                                                                                        
%Pfreq(Residential_Status);                                                                                                             
%Pfreq(Is_Good);                                                                                                                        
%Pfreq(Number_Creditcard);                                                                                                              
%Pfreq(Have_Card);                                                                                                                      
%Pfreq(Have_Web_Login);                                                                                                                 
%Pfreq(Have_Car);                                                                                                                       
%Pfreq(Occupation_Type);                                                                                                                
%Pfreq(Income_Group);                                                                                                                   
%Pfreq(Qualification);                                                                                                                  
%Pfreq(Vintage);                                                                                                                        
%Pfreq(Gender);                                                                                                                         
%Pfreq(Bank_Accounts);                                                                                                                  
%Pfreq(Pancard_Ind);                                                                                                                    
                                                                                                                                        
/*----------------------------------------------------------------------------------------------------------*/                          
                                                                                                                                        
/*for continuos variables*/                                                                                                             
%macro Uni(Var);                                                                                                                        
   proc univariate data = in.Data_Logistic;                                                                                             
   Var &var.;                                                                                                                           
   run;                                                                                                                                 
%mend Uni;                                                                                                                              
                                                                                                                                        
%Uni(Credit_Limit);                                                                                                                     
%Uni(Income_other_Source);                                                                                                              
                                                                                                                                        
/*----------------------------------------------------------------------------------------------------------*/                          
                                                                                                                                        
/*for discrete variable calculating woe*/                                                                                               
                                                                                                                                        
proc sql;                                                                                                                               
  /*create table tt as */                                                                                                               
  select                                                                                                                                
  sum( case when Is_Good eq 1 then 1 else 0 end),                                                                                       
  sum( case when Is_Good eq 0 then 1 else 0 end) into :tot_good,:tot_bad                                                                
  from in.Data_Logistic;                                                                                                                
quit;                                                                                                                                   
                                                                                                                                        
%put &tot_good. &tot_bad;                                                                                                               
                                                                                                                                        
/*                                                                                                                                      
proc sql;                                                                                                                               
  create table tt as                                                                                                                    
  select                                                                                                                                
  sum( case when Is_Good eq 1 then 1 else 0 end) as tot_good,                                                                           
  sum( case when Is_Good eq 0 then 1 else 0 end) as tot_bad                                                                             
  from in.Data_Logistic;                                                                                                                
quit;                                                                                                                                   
*/                                                                                                                                      
                                                                                                                                        
proc sql;                                                                                                                               
   create table Residential_Status_tt as                                                                                                
   select  Residential_Status,                                                                                                          
   sum( case when Is_Good eq 1 then 1 else 0 end) as num_goods,                                                                         
   sum( case when Is_Good eq 0 then 1 else 0 end) as num_bads,                                                                          
                                                                                                                                        
   calculated num_goods/&tot_good as percent_goods,                                                                                     
   calculated num_bads/&tot_bad   as percent_bads,                                                                                      
                                                                                                                                        
   log(calculated percent_goods/calculated percent_bads) as woe_var,                                                                    
   (calculated percent_goods - calculated percent_bads)  as diff_goods_bads,                                                            
   calculated diff_goods_bads * calculated woe_var       as IV_var                                                                      
                                                                                                                                        
   from in.Data_Logistic                                                                                                                
   group by Residential_Status;                                                                                                         
quit;                                                                                                                                   
                                                                                                                                        
proc export data = Residential_Status_tt                                                                                                
outfile="D:\HARD DISK 2\ANAKEN STUDY MATERIAL\SAS\CANDRAGSHU CHACRABARTY\bach_ba1\SAS_Export\Residential_Status_tt.xls"                 
dbms= excel replace; run;                                                                                                               
                                                                                                                                        
                                                                                                                                        
                                                                                                                                        
* %good=nums_good/Total_Good, %bad=nums_bad/Total_bad                                                                                   
  Weight of Evidence = WOE = log(%good / %bad)                                                                                          
  Information Value  = IV = log(%good / %bad) * (%good - %bad) = WOE*(%good - %bad) = WOE*Diff_Good_Bad                                 
;                                                                                                                                       
                                                                                                                                        
                                                                                                                                        
%macro woe(var);                                                                                                                        
  proc sql;                                                                                                                             
    /*create table tt as */                                                                                                             
    select                                                                                                                              
    sum( case when Is_Good eq 1 then 1 else 0 end),                                                                                     
    sum( case when Is_Good eq 0 then 1 else 0 end) into :tot_good,:tot_bad                                                              
    from in.Data_Logistic;                                                                                                              
  quit;                                                                                                                                 
                                                                                                                                        
  proc sql;                                                                                                                             
    create table &var._tt as                                                                                                            
    select  &var.,                                                                                                                      
    sum( case when Is_Good eq 1 then 1 else 0 end) as num_goods,                                                                        
    sum( case when Is_Good eq 0 then 1 else 0 end) as num_bads,                                                                         
                                                                                                                                        
    calculated num_goods/&tot_good as percent_goods,                                                                                    
    calculated num_bads /&tot_bad  as percent_bads,                                                                                     
                                                                                                                                        
    log(calculated percent_goods/calculated percent_bads) as woe_var,                                                                   
    (calculated percent_goods - calculated percent_bads)  as diff_goods_bads,                                                           
    calculated diff_goods_bads * calculated woe_var       as IV_var                                                                     
                                                                                                                                        
    from in.Data_Logistic                                                                                                               
    group by &var.;                                                                                                                     
  quit;                                                                                                                                 
%mend woe;                                                                                                                              
                                                                                                                                        
%woe(Residential_Status);                                                                                                               
%woe(Qualification);                                                                                                                    
                                                                                                                                        
/*                                                                                                                                      
%macro woe(var);                                                                                                                        
  proc sql;                                                                                                                             
    *create table tt as ;                                                                                                               
    select                                                                                                                              
    sum( case when Is_Good eq 1 then 1 else 0 end),                                                                                     
    sum( case when Is_Good eq 0 then 1 else 0 end) into :tot_good,:tot_bad                                                              
    from in.Data_Logistic;                                                                                                              
  quit;                                                                                                                                 
                                                                                                                                        
  proc sql;                                                                                                                             
    create table &var._tt as                                                                                                            
    select  &var.,                                                                                                                      
    sum( case when Is_Good eq 1 then 1 else 0 end) as num_goods,                                                                        
    sum( case when Is_Good eq 0 then 1 else 0 end) as num_bads,                                                                         
                                                                                                                                        
    calculated num_goods/&tot_good as percent_goods,                                                                                    
    calculated num_bads /&tot_bad  as percent_bads,                                                                                     
                                                                                                                                        
    log(calculated percent_goods/calculated percent_bads) as woe_var,                                                                   
    (calculated percent_goods - calculated percent_bads)  as diff_goods_bads,                                                           
    calculated diff_goods_bads * calculated woe_var       as IV_var                                                                     
                                                                                                                                        
    from in.Data_Logistic                                                                                                               
    group by &var.;                                                                                                                     
  quit;                                                                                                                                 
                                                                                                                                        
    proc export data = &var._tt                                                                                                         
    outfile="D:\HARD DISK 2\Biswarup\Project\Export\&var._tt.xls"                          
    dbms= excel replace; run;                                                                                                           
%mend woe;                                                                                                                              
                                                                                                                                        
%woe(Residential_Status);                                                                                                               
%woe(Is_Good);                                                                                                                          
%woe(Number_Creditcard);                                                                                                                
%woe(Have_Web_Login);                                                                                                                   
%woe(Have_Car);                                                                                                                         
%woe(Occupation_Type);                                                                                                                  
%woe(Income_Group);                                                                                                                     
%woe(Qualification);                                                                                                                    
%woe(Vintage);                                                                                                                          
%woe(Gender);                                                                                                                           
%woe(Bank_Accounts);                                                                                                                    
%woe(Pancard_Ind);                                                                                                                      
                                                                                                                                        
*/                                                                                                                                      
                                                                                                                                        
/*----------------------------------------------------------------------------------------------------------*/                          
                                                                                                                                        
/*for continuous variables*/                                                                                                            
                                                                                                                                        
proc rank data=in.Data_Logistic groups=10 out=project;                                                                                  
   var Credit_Limit;                                                                                                                    
   ranks Credit_limit_Rank; *--> 'Credit_Limit_Rank' is the new var. in the 'Project' dataset;                                          
run;                                                                                                                                    
                                                                                                                                        
proc freq data=project;                                                                                                                 
table credit_limit_rank;run;                                                                                                            
                                                                                                                                        
                                                                                                                                        
%macro woe_c(var);                                                                                                                      
  proc sql;                                                                                                                             
    /*create table tt as */                                                                                                             
    select                                                                                                                              
    sum( case when Is_Good eq 1 then 1 else 0 end),                                                                                     
    sum( case when Is_Good eq 0 then 1 else 0 end) into :tot_good,:tot_bad                                                              
    from in.Data_Logistic;                                                                                                              
  quit;                                                                                                                                 
                                                                                                                                        
  proc rank data=in.Data_Logistic groups=10 out=project_&var.;                                                                          
    var &var.;                                                                                                                          
    ranks &var._Rank;                                                                                                                   
  run;                                                                                                                                  
                                                                                                                                        
  proc sql;                                                                                                                             
    create table &var._rank as                                                                                                          
                                                                                                                                        
    select  &var._rank,                                                                                                                 
            min(&var.) as &var._min,                                                                                                    
            max(&var.) as &var._max,                                                                                                    
    sum( case when Is_Good eq 1 then 1 else 0 end) as num_goods,                                                                        
    sum( case when Is_Good eq 0 then 1 else 0 end) as num_bads,                                                                         
                                                                                                                                        
    calculated num_goods/&tot_good as percent_goods,                                                                                    
    calculated num_bads /&tot_bad  as percent_bads,                                                                                     
                                                                                                                                        
    log(calculated percent_goods/calculated percent_bads) as woe_var,                                                                   
    (calculated percent_goods - calculated percent_bads)  as diff_goods_bads,                                                           
    calculated diff_goods_bads * calculated woe_var       as IV_var                                                                     
                                                                                                                                        
    from project_&var.                                                                                                                  
    group by &var._rank;                                                                                                                
  quit;                                                                                                                                 
                                                                                                                                        
    proc export data = &var._rank                                                                                                       
    outfile="E:\SAS\CANDRAGSHU CHACRABARTY\bach_ba1\New folder\&var._rank.xls"                                                          
    dbms= excel replace; run;                                                                                                           
%mend woe_c;                                                                                                                            
                                                                                                                                        
%woe_c(Credit_Limit);                                                                                                                   
%woe_c(Income_Other_Source);                                                                                                            
                                                                                                                                        
* Weight of Evidence = woe= log(%good / %bad)                                                                                           
  Information Value  = IV = log(%good / %bad) * (%good - %bad) = WOE*(%good - %bad) = WOE*Diff_Good_Bad                                 
IV<0.1 => reject the var., 0.1<IV<0.3 then very poor preditive power, 0.3<IV<1 => poor predictive power                                 
IV>3 => highly predictive lead to biasness of the estimator,                                                                            
;                                                                                                                                       
                                                                                                                                        
/*----------------------------------------------------------------------------------------------------------*/                          
                                                                                                                                        
/* introducing woe variable corresponding to each variables */                                                                          
data model;                                                                                                                             
 set in.Data_Logistic;                                                                                                                  
                                                                                                                                        
 if Qualification = 'Grad' then qualification_woe = -0.039431176;                                                                       
 else                           qualification_woe = 0.986843553;                                                                        
                                                                                                                                        
 if Have_Car = 0 then Have_car_woe = -0.266152178;                                                                                      
 else                 Have_car_woe = 0.367547898;                                                                                       
                                                                                                                                        
 if Pancard_Ind = 0 then Pancard_Ind_woe = -0.067225888;                                                                                
 else                    Pancard_Ind_woe = 0.506695879;                                                                                 
                                                                                                                                        
 if Have_Web_Login = 0 then Have_Web_Login_woe = -0.208502522;                                                                          
 else                       Have_Web_Login_woe = 0.452221973;                                                                           
                                                                                                                                        
 if Vintage = 'New' then Vintage_woe = -0.311935705;                                                                                    
 else                    Vintage_woe = 0.316643205;                                                                                     
                                                                                                                                        
 if      Number_Creditcard = 1 then Number_Creditcard_woe = -0.367775291;                                                               
 else if Number_Creditcard = 2 then Number_Creditcard_woe = -0.271999737;                                                               
 else if Number_Creditcard = 3 then Number_Creditcard_woe = -0.106613934;                                                               
 else                               Number_Creditcard_woe = 0.24735815;                                                                 
                                                                                                                                        
 if residential_status = 'Own' then residential_status_woe = 0.138072003;                                                               
 else                               residential_status_woe = -0.206781623;                                                              
                                                                                                                                        
       if 4790  ge Credit_Limit le 9200  then CL_woe=-1.0252;                                                                           
  else if 9204  ge Credit_Limit le 11599 then CL_woe=-0.6530;                                                                           
  else if 11600 ge Credit_Limit le 13500 then CL_woe= -0.462;                                                                           
  else if 13502 ge Credit_Limit le 15200 then CL_woe= -0.362;                                                                           
  else if 15204 ge Credit_Limit le 17489 then CL_woe= -0.211;                                                                           
  else if 17490 ge Credit_Limit le 26900 then CL_woe= -0.04;                                                                            
  else if 26910 ge Credit_Limit le 34500 then CL_woe= 0.349;                                                                            
  else                                        CL_woe= 1.994;                                                                            
run;                                                                                                                                    
                                                                                                                                        
                                                                                                                                        
/* spliting sample into devlopment sample and validating sample */                                                                      
proc surveyselect data = model                                                                                                          
                  out = split                                                                                                           
                  samprate = .7 seed=1234                                                                                               
                  outall;                                                                                                               
run;                                                                                                                                    
*new dataset will be created of named 'split' that will be exactly same as 'model' rather a var. will be there of named                 
'selection indicator' will take 1 if selected in the sample other wise 0.                                                               
;                                                                                                                                       
                                                                                                                                        
                                                                                                                                        
/* creating devlopment sample and validation sample */                                                                                  
data training                                                                                                                           
     validation;                                                                                                                        
   set split;                                                                                                                           
   if selected =1 then output training;                                                                                                 
   else output                validation;                                                                                               
run;  *--> traning=devlopment sample, validation=validation sample;                                                                     
                                                                                                                                        
                                                                                                                                        
/* 'proc reg' is imp for 'vif' for all the explanatory var. */                                                                          
proc reg data=training;                                                                                                                 
  model Is_Good = qualification_woe                                                                                                     
                  Have_car_woe                                                                                                          
                  Have_Web_Login_woe                                                                                                    
                  Vintage_woe                                                                                                           
                  Number_Creditcard_woe                                                                                                 
                  residential_status_woe                                                                                                
                  CL_woe / vif ;                                                                                                        
run; quit;                                                                                                                              
*--> VIF=1/(1-Rj^2),where Rj^2 is R^2 in the reg of Xj on the remaining (k-2) explanatory var. out of (k-1) explanatory                 
var in the model ;                                                                                                                      
                                                                                                                                        
/* 'proc logistic' for logistic reg. */                                                                                                 
proc logistic data=training  descending;                                                                                                
  model Is_Good = qualification_woe                                                                                                     
                  Have_car_woe                                                                                                          
                  Have_Web_Login_woe                                                                                                    
                  Vintage_woe                                                                                                           
                  Number_Creditcard_woe                                                                                                 
                  residential_status_woe                                                                                                
                  CL_woe / lackfit                                                                                                      
                                                                                                                                        
  /*selection=stepwise                                                                                                                  
        ctable pprob=(0 to 1 by .1)                                                                                                     
        slentry=0.05                                                                                                                    
        slstay=0.05                                                                                                                     
        clodds=both                                                                                                                     
        clparm=both                                                                                                                     
        lackfit */ ;                                                                                                                    
        output out=probs predicted=phat;                                                                                                
run; *--> here we hv taken 1 as 'good' and 0 as 'bad' but by default SAS will take 0 as 'good' and 1 as 'bad' that's why                
'descending' is imp.;                                                                                                                   
                                                                                                                                        
                                                                                                                                        
/*                                                                                                                                      
 proc logistic data=training  descending;                                                                                               
  model Is_Good = qualification_woe                                                                                                     
                  Have_car_woe                                                                                                          
                  Have_Web_Login_woe                                                                                                    
                  Vintage_woe                                                                                                           
                  Number_Creditcard_woe                                                                                                 
                  residential_status_woe                                                                                                
                  CL_woe/lackfit                                                                                                        
                                                                                                                                        
        ctable pprob=(0 to 1 by .1)                                                                                                     
        slentry=0.05                                                                                                                    
        slstay=0.05                                                                                                                     
        clodds=both                                                                                                                     
        clparm=both                                                                                                                     
        lackfit ;                                                                                                                       
        output out=probs predicted=phat;                                                                                                
run                                                                                                                                     
*/                                                                                                                                      
                                                                                                                                        
/*-------------------------------------------------------------------------------------------------------*/                             
                                                                                                                                        
proc sql;                                                                                                                               
  select                                                                                                                                
  sum( case when Is_Good eq 1 then 1 else 0 end),                                                                                       
  sum( case when Is_Good eq 0 then 1 else 0 end) into :tot_good,:tot_bad                                                                
  from training;                                                                                                                        
quit;                                                                                                                                   
                                                                                                                                        
                                                                                                                                        
proc rank data = probs out = logistic_probs group = 10;                                                                                 
   var phat;                                                                                                                            
   ranks r_phat;                                                                                                                        
run;                                                                                                                                    
                                                                                                                                        
                                                                                                                                        
proc freq data=logistic_probs;                                                                                                          
table r_phat; run;                                                                                                                      
                                                                                                                                        
                                                                                                                                        
proc sql;                                                                                                                               
    create table KS_cal as                                                                                                              
    select r_phat,                                                                                                                      
      sum(case when is_good eq 0 then 1 else 0 end) as bad,                                                                             
      sum(case when is_good eq 1 then 1 else 0 end) as good                                                                             
    from logistic_probs                                                                                                                 
    group by r_phat;                                                                                                                    
                                                                                                                                        
         data ks_cal;                                                                                                                   
            set ks_cal;                                                                                                                 
               if first.r_phat then cg = 0;                                                                                             
                                    cg + good;                                                                                          
               if first.r_phat then cb = 0;                                                                                             
                                    cb + bad;                                                                                           
               output;                                                                                                                  
         run;                                                                                                                           
                                                                                                                                        
         data ks_cal;                                                                                                                   
            set ks_cal;                                                                                                                 
               percent_cg = (cg/&tot_good.) * 100;                                                                                      
               percent_cb = (cb/&tot_bad.) * 100;                                                                                       
               diff = abs(percent_cg - percent_cb);                                                                                     
               bad_rate = bad/(bad + good);                                                                                             
         run;                                                                                                                           
quit;                                                                                                                                   
                                                                                                                                        
/*                                                                                                                                      
proc sql;                                                                                                                               
    create table KS_cal_2 as                                                                                                            
    select r_phat,                                                                                                                      
      sum(case when is_good eq 0 then 1 else 0 end) as bad,                                                                             
      sum(case when is_good eq 1 then 1 else 0 end) as good                                                                             
    from logistic_probs                                                                                                                 
    group by r_phat;                                                                                                                    
quit;                                                                                                                                   
                                                                                                                                        
data ks_cal_2;                                                                                                                          
  set ks_cal_2;                                                                                                                         
  if first.r_phat then cg = 0;                                                                                                          
                    cg + good;                                                                                                          
  if first.r_phat then cb = 0;                                                                                                          
                     cb + bad;                                                                                                          
   output;                                                                                                                              
run;                                                                                                                                    
                                                                                                                                        
data ks_cal_2;                                                                                                                          
     set ks_cal_2;                                                                                                                      
     percent_cg = (cg/&tot_good.) * 100;                                                                                                
     percent_cb = (cb/&tot_bad.) * 100;                                                                                                 
     diff = abs(percent_cg - percent_cb);                                                                                               
     bad_rate = bad/(bad + good);                                                                                                       
run;                                                                                                                                    
                                                                                                                                        
*/                                                                                                                                      
                                                                                                                                        
                                                                                                                                        
proc sql;                                                                                                                               
    create table accur_cal as                                                                                                           
    select r_phat,                                                                                                                      
      min(phat) as min_prob,                                                                                                            
      max(phat) as max_prob,                                                                                                            
      mean(calculated min_prob, calculated max_prob) as mean_prob,                                                                      
      sum(case when is_good eq 0 then 1 else 0 end)  as bad,                                                                            
      sum(case when is_good eq 1 then 1 else 0 end)  as good,                                                                           
                                                                                                                                        
      log(calculated good/calculated bad)                  as actual,                                                                   
      log(calculated mean_prob/(1 - calculated mean_prob)) as predicted                                                                 
     from logistic_probs                                                                                                                
     group by r_phat;                                                                                                                   
quit;                                                                                                                                   
                                                                                                                                        
                                                                                                                                        
                                                                                                                                        
proc export data=work.accur_cal                                                                                                         
       outfile='D:\HARD DISK 2\Biswarup\Project\gini.xls'                                      
       dbms=excel2000                                                                                                                   
       replace;                                                                                                                         
run;                                                                                                                                    
                                                                                                                                        
 /*-----------------------------------------------------------------------------------------------------------------------*/            
                                                                                                                                        
  /* Validating model on test sample */                                                                                                 
proc sql;                                                                                                                               
  /*create table tt as */                                                                                                               
  select                                                                                                                                
  sum( case when Is_Good eq 1 then 1 else 0 end),                                                                                       
  sum( case when Is_Good eq 0 then 1 else 0 end) into :tot_good,:tot_bad                                                                
  from validation;                                                                                                                      
quit;                                                                                                                                   
                                                                                                                                        
                                                                                                                                        
data validation ;                                                                                                                       
    set validation;                                                                                                                     
    drop income_other_source_woe credit_limit_woe;                                                                                      
      log_odds =  -0.8753                                                                                                               
                   + 0.8403 * CL_woe                                                                                                    
                   + /*0.8069*income_other_source_woe*/                                                                                 
                   +0.6041 * qualification_woe                                                                                          
                   +0.4949 * Have_car_woe                                                                                               
                   +0.5823 * Pancard_Ind_woe                                                                                            
                   +0.4369 * Have_Web_Login_woe                                                                                         
                   +0.5504 * Vintage_woe                                                                                                
                   +0.6456 * Number_Creditcard_woe                                                                                      
                   +0.3112 * residential_status_woe;                                                                                    
    prob = exp(log_odds) / (1+exp(log_odds));                                                                                           
run;                                                                                                                                    
                                                                                                                                        
                                                                                                                                        
proc rank data = validation  out = validation_rank  group = 10;                                                                         
    var prob;                                                                                                                           
    ranks r_phat;                                                                                                                       
run;                                                                                                                                    
                                                                                                                                        
                                                                                                                                        
proc sql;                                                                                                                               
    create table KS_cal_val as                                                                                                          
    select r_phat,                                                                                                                      
      sum(case when is_good eq 0 then 1 else 0 end) as bad,                                                                             
      sum(case when is_good eq 1 then 1 else 0 end) as good                                                                             
    from validation_rank                                                                                                                
    group by r_phat;                                                                                                                    
                                                                                                                                        
         data ks_cal_val;                                                                                                               
            set ks_cal_val;                                                                                                             
               if first.r_phat then cg = 0;                                                                                             
                                    cg + good;                                                                                          
               if first.r_phat then cb = 0;                                                                                             
                                    cb + bad;                                                                                           
               output;                                                                                                                  
         run;                                                                                                                           
                                                                                                                                        
         data ks_cal_val;                                                                                                               
            set ks_cal_val;                                                                                                             
               percent_cg = (cg/&tot_good.) * 100;                                                                                      
               percent_cb = (cb/&tot_bad.) * 100;                                                                                       
               diff = abs(percent_cg - percent_cb);                                                                                     
               ro = bad/(bad+good);                                                                                                     
          run;                                                                                                                          
quit;                                                                                                                                   
                                                                                                                                        
                                                                                                                                        
proc sql;                                                                                                                               
    create table acc_cal_val as                                                                                                         
    select r_phat,                                                                                                                      
      min(prob) as min_prob,                                                                                                            
      max(prob) as max_prob,                                                                                                            
      mean(calculated min_prob,calculated max_prob) as mean_prob,                                                                       
      sum(case when is_good eq 0 then 1 else 0 end) as bad,                                                                             
      sum(case when is_good eq 1 then 1 else 0 end) as good,                                                                            
                                                                                                                                        
      log(calculated good/calculated bad)                  as actual,                                                                   
      log(calculated mean_prob/(1 - calculated mean_prob)) as predicted                                                                 
    from validation_rank                                                                                                                
    group by r_phat;                                                                                                                    
quit;

proc export data=work.acc_cal_val                                                                                                       
       outfile='D:\HARD DISK 2\Biswarup\Project\gini_val.xls'                                  
       dbms=excel2000                                                                                                                   
       replace;                                                                                                                         
run;  

proc export data=work.accur_cal                                                                                                         
       outfile='D:\HARD DISK 2\Biswarup\Project\gini.xls'                                      
       dbms=excel2000                                                                                                                   
       replace;                                                                                                                         
run;                                                                                                                                   
                                                                                                                                        
                                                                                                                                                              

                                                                                                             
                                                                                                                                   
                                                                                                                                        
                                                                                                                                        
                                                                                                                                        



