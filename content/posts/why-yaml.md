+++ 
draft = false
date = 2019-06-11T08:53:47-07:00
title = "Why Yaml For Configuration Management"
description = ""
slug = "" 
tags = ["yaml", "json", "configuration management"]
categories = []
externalLink = ""
series = []
+++
# What is Yaml

YAML is a human friendly data serialization standard for all programming languages. [Official page](https://yaml.org/)

# Yaml is simple

* It is human readalbe

* Simple to [learn](https://learnxinyminutes.com/docs/yaml/)

* Allow comments (I am looking at you, json)

* Allow reference for re-use (I am looking at you again, json)

# Yaml is not turning complete 

* Yaml is not turning complete and is hence not "code": you cannot program it, no branching, no function call... It therefore enforces a strict seperation of concerns between declaration of intention and execution of intented functionality see more dicussion [here](https://news.ycombinator.com/item?id=16238005)

# Yaml can be compared programmtically
* for example [structurediff](https://pypi.org/project/structurediff/)
* We can normalize any yaml configuration and calculate hash/digest of it
    * Library to normalize yaml file and compute consistently hash/digest
* json as subset of yaml can be compared programmatically as well