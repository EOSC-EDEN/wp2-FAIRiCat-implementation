wp2-FAIRiCat-implementation; an example
=======================================

While implementing FAIRiCat for our repositories (Dataverse archives) it became apparent that the way we solved some of the issues we encountered might be helpful for others that would need to implement FAIRiCat for their repositories. 

Information about FAIRiCat can be found here: https://signposting.org/FAIRiCat/
And the 'Linksets' format i is based on (yes it is about links): https://www.rfc-editor.org/rfc/rfc9264.html

Decisions made up front
--------------------------

- Support the catalog part and not the objects part. 
  The objects, in our case Datasets, already have exposure of metadata in several formats and also we support signposting (Schema.org). 
  The main omission in our machine readable information exposure it the repository level information. 

- Implement the 'JSON file' and 'well-known-uri' based approach. 
  In this way we keep it separate from the repository application (Dataverse), because  we do not need to change it's HTTP header response. 


The 'Linkset' Links
-------------------

## OAI-PMH

## Sitemap

## OpenAPI

## Robots.txt

## File formats

## Licenses


Appendix
--------

As example the template file (Jinja2) and Ansible code fragments that we used to deploy it. 
The main ansible task is in file `main.yml` and the template in `index.json.j2`. 
Note that we also have the variable `shared_hostname` defined for each of our archives.  

