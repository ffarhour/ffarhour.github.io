# Cool things about FFMade.me
I wanted to list down the cool things I've done with this website, so here it is:

## 1. Jekyll site hosted on github
This website uses the Jekyll engine, and is hosted by github, under their github pages service.

## 2. Pre-Commit Script
I've made a pre-commit script for this website, which:
 - automatically looks at a csv file containing a list of other git repositories
 - fetches their docs folders
 - re-names and adds relevant formatting information
 - creates permalinks for each separate docs folder

And all this allows Jekyll to automatically generate webpages from markdown files in all the different repositories. This of course makes it a lot easier for me to collate my projects and their docs in one place.

### Pre-Commit Script
This is a rather simple script that automates the above processes mentioned.
```
###
### The following block runs after commit to "master" branch
###
if [ `git rev-parse --abbrev-ref HEAD` == "master" ]; then

    # Layout prefix is prepended to each markdown file synced
    ###################################################################
    LAYOUT_PREFIX_s='---\r\nlayout: index' #start
    LAYOUT_PREFIX_e='\r\n---\r\n\r\n' #end
    LAYOUT_PREFIX=$LAYOUT_PREFIX_s
    # checkout all files in docs folders from specified repos and put them
    # in projects folder
    ###################################################################
    while IFS=, read col1
    do
        NAME=$(echo ${col1##*/}| cut -d'.' -f 1) #.md
        git fetch $col1 && git checkout FETCH_HEAD -- docs #README.MD
        #echo -e $LAYOUT_PREFIX >> projects/$NAME
        FILES=docs/*
        for file in $FILES
        do
            filename="${file##*/}"
            filename="${filename%.*}"
            LAYOUT_PREFIX+="\r\ntitle: $NAME\r\npermalink: projects/$NAME/$filename.html\r\nsections:\r\n    github: $col1 $LAYOUT_PREFIX_e"
            echo -e $LAYOUT_PREFIX | cat - "$file" > temp && mv temp "$file"
            LAYOUT_PREFIX=$LAYOUT_PREFIX_s
        done
        rm -rf projects/$NAME
        mkdir projects/$NAME
        cp docs/. projects/$NAME/ -rfv
        rm -rf docs
        echo "projects/$NAME created"
    done < checkouts.csv
    git add --all
    #git commit -m "Synced readme.md docs from specified repos"
fi
```
 I will need to give credit to this stack overflow post & answer, which helped a lot in making the above simple script: http://stackoverflow.com/questions/15214762/how-can-i-sync-documentation-with-github-pages
