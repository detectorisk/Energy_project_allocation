# Energy_project_allocation
The aim is to construct renewable energy sites out of a pool of 50 simulated sites by identifying those that can be procured based on available staffing resources. The simulated potential sites, and resulting staff hours data was created with the help of Chatgpt4. 
By implementing a linear optimization approach, the analysis recommends the acquisition of 22 sites.

# My baseline case study
I initially identified 50 potential sites that can be acquired for renewables production, amounting to a total of 2,382 MW production of electricity by April 2032 (considering a standard of 9 years for the project to be set up). Among the sites thus identified, the largest electricity producers in terms of MW are windmills, while BESS accounts for the smallest producer segments. However, I posit the production of renewables in all these 50 sites at the same time is an unrealistic target to be achieved in this period, given both staffing and programmatic constraints. 

To begin with, there is a huge gap between staff required and present staff numbers to meet the demands of all these sites in the next 9 years. While this could be a factor toward initiating recruitment drives, I find myself less inclined to ask of the company that it hires more people immediately to meet the demands of the staffing requirements of all 50 sites. This is evident since the staff expansion required is to degrees much higher than the present staff numbers. Especially in the case of managerial roles the staffing hours deficit is about three times if we were to pursue all 50 sites at the same time.  

Thus, my focus shifted toward looking at an optimal solution first based on present staff numbers and using that as a baseline for future development of projects. Furthermore, I believe out optimization methodology can be used company wide, to account for project feasibility, for staffing resource optimization, and in choosing long term projects. 

# Our Methodology 

I have already established the potential energy capacity that can be produced out of each of the renewable site, and the staffing hours required out of each of the roles for each of the different technologies, namely BESS SPP, Solar, Wind plants with less than 50 MW capacity, and Wind plants with more than 50 MW capacity. Here, the staffing requirements is only a function of the type of technology employed, and unaffected by the size of the project. Interestingly, I find that the number of managerial hours required in Wind projects is much more than for Solar and BESS (see Figure 7). Wind technologies with more than 50 MW capacity demand the most staff hours from all job roles, while I have also seen they are among the highest electricity producers. Additionally, taking energy capacity into our model at this stage is counterintuitive, as we won’t be able to do linear programming (since for the same staffing inputs for each technology, given everything else as constant, our data suggests different outputs in terms of energy production). Thus, I will account for energy capacities at a later stage in the model. 

Notably, I assume here that the only cost of production to get the site running is that of staff salaries and a premium of 500k for site acquisition. Both can be taken as fixed costs and are thus taken out of the model. Any new hire of staff will be a variable cost (especially since there will be cost to training and pulling up slack on an incumbent employee etc.). Given these staffing resources, and a fixed cost of 500k for each site to be acquired, an output maximizing function will essentially be a question of how many sites of each technology we would like to invest in. Thus, the maximization function would be: 
f(x,y,z,w)=x+y+z+w
where,
x is the number of sites of BESS SPP; y is the number of sites of Solar; z is the number of sites of wind MW<50; w is the number of sites of wind MW>=50

and the staffing hours constraints are :

For BESS: 
(Total staff hours available with team)-x(Total staff hours required from team for 1 BESS project)≥0

For Solar: 
[(Total staff hours available with team)-x(Total staff hours required from team for 1 BESS project)]- y((Total staff hours required from team for 1 Solar project)≥0

For Wind <50 MW: 
[(Total staff hours available from team)-x(Total staff hours required from team for 1 BESS project)-y(Total staff hours required from team for 1 Solar project)]-z(Total staff hours required from team for 1 Wind<50 project 

For Wind >=50 MW: 
[(Total staff hours available from team)-x(Total staff hours required from team for 1 BESS project)-y(Total staff hours required from team for 1 Solar project)-z(Total staff hours required from team for 1 Wind<50 project]-w(Total staff hours required from team for 1 Wind≥50 project)  

and other constraints: 
x≤6; y≤11; z≤21; w<-12

# Optimal solution 

Using a linear optimization model, I found that the optimal number of renewables sites came to be 22, given the staffing available for the next 9 years. The model suggests that we invest in 5 BESS SPP projects, 11 Solar projects, 6 Wind <50 MW projects, and 0 Wind>=50 MW projects. I realize, the last result is thus because large wind projects demand a lot of staff across various roles, and thus an optimal solution would be one that has mixed bag of projects with fewer massive projects driving the output. This can also be seen as a drawback to our analysis, as I haven’t focused on maximizing energy production here, but rather focused only on maximizing number of projects, to have a diverse pool of sites, thus mitigating for project related risks. Through this this model's output, we get a total electricity production of 904 MW.

Note, we had not specified in our model to take at least one site per technology. When I tried giving another set of constraint to this effect, I found that the model re-allocated the staff from one of the Wind<50 MW sites to build the Wind>=50 MW site. This perhaps is the best I can do now, given the staffing list. The 22 projects thus got will give us a total electricity production of 1,059 MW. 
