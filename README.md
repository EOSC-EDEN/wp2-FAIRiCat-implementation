wp2-FAIRiCat-implementation; an example
=======================================

While implementing FAIRiCat for our repositories (Dataverse archives at DANS) it became apparent that the way we solved some of the issues we encountered might be helpful for others that would need to implement FAIRiCat for their repositories. 

Information about FAIRiCat can be found here: https://signposting.org/FAIRiCat/
And the 'Linksets' format, which it is based on (it is about links): https://www.rfc-editor.org/rfc/rfc9264.html


Decisions made up front
-----------------------

Decisions we made before starting to implement. 

- Support the catalog part (Repository-level affordances) and not the objects part. 
  The main omission in our machine readable information exposure it the repository level information.
  The objects, in our case Datasets, already have exposure of metadata in several formats and also we support signposting (Schema.org). Dataset landing pages can be found via Sitemaps and via OAI-PMH. 

- Implement the 'JSON file' and 'well-known-uri' based approach. 
  In this way we keep it separate from the repository application (Dataverse), because we do not need to change it's HTTP header response. Implementation can be done without changing application code. 
  Systems that try to discover our FAIRiCat services information must 'probe' for that uri at `.well-known/api-catalog`. This information will not be exposed via the HTTP response header of the 'homepage' or 'root' of the site.  


The 'Linkset' Links
-------------------

The JSON fragments shown here are mainly based on the example in the FAIRiCat webpage. The repository used as example is our DANS archive at `https://ssh.datastations.nl`. 


## OAI-PMH

This is the first example that FAIRiCat mentions and for a good reason. 
For this 'Link' we need to know the URL of the Repository, Dataverse in our case. This is `/oai`, so not `oai-pmh`. The JSON fragment for our SSH archive looks like this: 

```
    {
      "anchor": "https://ssh.datastations.nl/oai",
      "service-doc": [
        {
          "href": "https://www.openarchives.org/OAI/openarchivesprotocol.html",
          "type": "text/html",
          "title": "Open Archives Initiative - Protocol for Metadata Harvesting - v.2.0"
        }
      ],
      "service-meta": [
        {
          "href": "https://ssh.datastations.nl/oai?verb=Identify",
          "type": "application/xml"
        }
      ]
    }
```

This shows that discovery of the OAI-PMH can be done by inspecting all the 'service-doc' items and look for the `https://www.openarchives.org/OAI/openarchivesprotocol.html`. Assuming every FAIRiCat implementing repository would use that for their OAI Link. The `service-meta` does not add much, `verb=Identify` is part of the OAI standard, but when unaware of the OAI standard it will give you some extra information. 


## Sitemap

Most repositories will have this in place in order to improve the efficiency of search engine 'crawlers'. Our Dataverse archive support this, so we can add it as a Link: 


```
    {
      "anchor": "https://ssh.datastations.nl/sitemap/sitemap.xml",
      "service-doc": [
        {
          "href": "https://www.sitemaps.org/protocol.html", 
          "type": "text/html",
          "title": "sitemaps.org - Protocol"
        }
      ]
    }
```

Without FAIRiCat, discovery of the sitemap would be done by looking for `/sitemap.xml` and `/sitemap/sitemap.xml`. Alternatively one could look into the `robots.txt`, as described further on. Although it does not seem to be an essential addition, we want to make our FAIRiCat as complete as simply possible. 


## OpenAPI

This is important, if you happen to have this (maybe even for multiple APIs) you should definitely add it. The only other way to discover these is via reading the repository systems documentation or code. Alternatively, one could try to find links via a search engine and or ask an AI. 
Dataverse fortunately exposes it's API (not certain if it is complete) at `/openapi`. 


```
    {
      "anchor": "https://ssh.datastations.nl/openapi",
      "service-desc": [
        {
          "href": "https://ssh.datastations.nl/openapi",
          "type": "application/yaml"
        }
      ],
      "service-doc": [
        {
          "href": "https://spec.openapis.org/oas/v3.0.3",
          "type": "text/html",
          "title": "OpenAPI Specification v3.0.3"
        }
      ]
    }
```

Note the version number in that specification, this did change with the Dataverse upgrades. It is therefore advisable to check this number in the openapi (yml) service description after upgrading the application that implements the API. This could be automated because it is part of the OpenAPI specification. 


## Robots.txt

Most repositories will have this in place, just like with the sitemap it could improve the efficiency of search engine 'crawlers' and lessen the burden of that crawling on your service. Probably the `robots.txt` file will have a reference to the sitemap, so crawlers could find the sitemap via the robots.txt. Having the `robots.txt` at the site root is pretty standard, but not mandatory. 

```
    {
      "anchor": "https://ssh.datastations.nl/robots.txt",
      "service-doc": [
        {
          "href": "https://datatracker.ietf.org/doc/rfc9309/",
          "type": "text/html",
          "title": "RFC 9309 - Robots Exclusion Protocol"
        }
      ]
    }
```


## File formats

The 'File formats' information should clarify what types of files (mime-type) are allowed and or preferred or recommended. We identified that this information is important for potential depositors to select an repository. 
As far as I know there is no standard for this kind of curation information. Therefore we are stuck with human readable information like HTML pages and or PDF documents. 
After some browsing and searching on our organization's website `https://dans.knaw.nl`, I discovered a webpage that served the purpose. 

```
    {
      "anchor": "https://dans.knaw.nl/en/file-formats/",
      "service-doc": [
        {
          "href": "https://dans.knaw.nl/en/file-formats/",
          "type": "text/html",
          "title": "File Formats"
        }
      ]
    }
```

Hopefully the site maintenance team respect the wish to have this URL semi-persistent. Because, if that URL changes we need to update the FAIRiCat JSON file respectively. 
Because there is no standard, it is not straightforward for a harvester of the FAIRiCat catalog to automatically detect the 'File format' information without prior information on how to detect it.  


## Licenses

The 'License' information should specify what licenses the archive supports. 
As with the file formats, potential depositors could use this to select a repository. 
This information is available via the API at `/api/licenses`. 

```
    {
      "anchor": "https://ssh.datastations.nl/api/licenses",
      "service-desc": [
        {
          "href": "https://ssh.datastations.nl/openapi",
          "type": "application/yaml"
        }
      ],
      "service-doc": [
        {
          "href": "https://spec.openapis.org/oas/v3.0.3",
          "type": "text/html",
          "title": "OpenAPI Specification v3.0.3"
        }
      ]
    }
```

The problem here is that there is no separate documentation or specification for this `licenses` API endpoint; so no `/api/licenses/openapi.yaml` or `/api/licenses/docs`. We can oly point at that overall API in the description and documentation. 



Issues 
------

Some practical issues and limitations encountered during the implementation of FAIRiCat.  

- When trying to imagine how 'harvesting' systems are going to use that FAIRiCat information to discover the services some limitation become apparent. When there is no specification for the information at the link (description for the anchor URL), then there is no easy way to automate the discovery. We have this problem with the 'File format' and 'Licenses' information. 

- Some Links are not as stable as we would like them to be; the 'File formats' again, but also the OpenAPI service description, which has a version number in the URL. Then there is that Sitemap that has a different filename depending on the number of datasets (research objects) in our archive. 

As a side effect of this work; by implementing FAIRiCat we are forced to discover the services that we have, and might not always be much aware of. One could even imagine that we have some kind of 'health' check that looks at all the 'anchors' to see if everything is up and running. 


## Possible improvements

Observations and suggestions for improvement.

 - Whenever the 'Linkset' contains something that is dependent on the repository content and changes over time, we could have a (nightly) process in place (crontab) that generates an updated version of the JSON file. That updating process would be similar to what is generally done for keeping the sitemap up to date. 
 For example, in our case, that sitemap filename depends on the total number of datasets in the Dataverse archive. The update script could determine this and change the name in that Link if needed. 

 - We could add 'Links' to other formats, like DCAT-AP, which also describe the services of the repository. Possibly that format improves the automatic discovery. 

- Not an improvement, but merely an observation. Full automated discovery of specific services might not be possible, but FAIRiCat will support semi-automatic discovery. As an example, a harvesting system could retrieve those links (from that Linkset) and then have a human determine what each link is about; is it 'Licensing' or 'File format' for instance. 

 
Appendix
--------

To make that JSON available the server must be configured. This appendix tries to explain how this was done on our (DANS) Dataverse archive system server. 


## Automated (ansible) configuration

As an example the template file (Jinja2) and Ansible code fragments that we use to deploy it. 
The main ansible task is in file `main.yml` and the template in `index.json.j2`. 
Note that we also have the variable `shared_hostname` defined for each of our archives. For our Social Sciences and Humanities (SSH) archive this hostname is `ssh.datastations.nl`. 

You can find the Ansible task file [main.yml](ansible/main.yml) and the Jinja2 template [index.json.j2](ansible/index.json.j2) in the `ansible` directory of this repository.

__NOTE__ at the time of writing FAIRiCat support is not in production yet!


## Configuration details

For those that are not familiar with Ansible code or want to try out some implementation directly, on  the commandline. 
The next description assumes you have access (and permissions) to the web server with your archival system on it. 

The Apache httpd service that we use on our archival system (with Dataverse and Payara) has a web root directory from where it will serve web content. Any HTML file placed there can in principle be made available via the web service. 
For getting the 'well known uri' approach working we create a directory with a SON file in it. 
In our case we have the web root at `/var/www/html`. 

```
cd /var/www/html
mkdir -p .well-known/api-catalog
touch .well-known/api-catalog/index.json
```

Now there is an `index.json` file, but it is empty. Copy the JSON that you have prepared before int the file or just copy the whole file, whatever works for you.

This file is then servers as `application/json` at the relative URL `/.well-known/api-catalog/index.json`

This is not enough for FAIRiCat support, the Apache configuration needs to be changed. 
When you have found the configuration file for Apache httpd this needs to be edited and the httpd service must be restarted for the changes to take effect. With our setup it is in one of the `*.conf` files in `/etc/httpd/conf.d/`  directory. 

```
      ProxyPassMatch ^/\.well-known/api-catalog !
      Alias /.well-known/api-catalog /var/www/html/.well-known/api-catalog/index.json
      <Directory /var/www/html/.well-known/api-catalog>
         Require all granted
         DirectoryIndex index.json
         Header set Link '<https://ssh.datastations.nl/.well-known/api-catalog>; rel="api-catalog"; type="linkset+json"; profile="https://signposting.org/FAIRiCat/"'
         <Files "index.json">
            ForceType application/linkset+json
         </Files>
      </Directory>
```

### Configuration explanation

Because we run the Dataverse web application with Payara and 'AJP' 
 via the Apache httpd service, we need to place that fragment in the configuration file before the 'AJP' line. We won't elaborate on 'AJP'; information about this setup can be found in the Dataverse installation guide. 

The first line in the fragment above; with `ProxyPassMatch` makes sure that `/.well-known/api-catalog` is not directed to the Dataverse web application, but instead, served from that web root; `/var/www/html`. 

The next line is an `Alias`, this makes sure that the `index.json` file is available via `/.well-known/api-catalog`. 

The `Directory` block has specifications for the content in `/var/www/html/.well-known/api-catalog`. 

By default a request for a directory will show the directory content, unless it has an `index.html` file, to make this work for `index.json` the `DirectoryIndex` has to be set. 

According to the FAIRiCat specification we have to set the HTTP response header, even if the JSON file is at the 'well known uri'. 
That `Header set Link` is doing that; specifying the relation, type and profile as required by FAIRiCat. 

Finally the `Files` block specifies that the content type of the `index.json` is not simply `application/json`, but `application/linkset+json`. 

