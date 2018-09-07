
#add them repository vao project hien tai, origin_clone dai dien cho repository nay.
`git remote add origin_clone ssh://git@bitbucket.org/thailt/vtvcab_bigdata_clone.git`

#set lai remote url cho repository origin
`git remote set-url origin ssh://git@bitbucket.org/thailt/vtvcab_bigdata_clone.git`

#clone code ve
`git clone ssh://git@bitbucket.org/thailt/vtvcab_bigdata_clone.git`


#push code len repository origin_clone, branch develop
`git push origin_clone develop`

#follow change of file: pom.xml
`git log --follow -p pom.xml`

#remove a remote 
`git remote rm origin_clone`
