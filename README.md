# Git-Project-Control
 Allows you to use the GIT control version system for projects (files / folders / documents) inside Windows Explorer













import os
import shutil
import ntsecuritycon    #
import win32security    #need install pywin32 module
from distutils.dir_util import copy_tree
import git
#####

#project_name = (input('Название проекта: ', ))
project_name = ('test')

#####

#####
DEVELOP = './PROJECT/DEVELOP_' + project_name
RELEASE = './PROJECT/RELEASE_' + project_name
SERVER = './PROJECT/SERVER_' + project_name
PROJECT = './PROJECT'
####

#####
try:
    server_repo = git.Repo.init('./PROJECT/SERVER_' + project_name, bare=True)    #git init new_repo(SERVER)

    hook_server = SERVER + '/hooks' + '/post_update'
    with open(hook_server, 'w') as update_hook:
        update_hook.write('#!/bin/sh\n')
        update_hook.write('cd ../' + project_name + '\n')
        update_hook.write('unset GIT_DIR\n')
        update_hook.write('git pull origin master\n')

        update_hook.write('DEV=`cat ../.develop.flag`\n')
        update_hook.write('cd ../' + project + '\n')
        update_hook.write('unset GIT_DIR' + '\n')
        update_hook.write('MAS=\\`git log | head -1 | awk \'{ print \\$2 }\'\\`' + '\n')
        update_hook.write('if [ $MAS ==  $DEV ]' + '\n')
        update_hook.write("then" + '\n')
        update_hook.write("cd .." + '\n')
        update_hook.write("unset GIT_DIR" + '\n')
        update_hook.write("git add ." + '\n')
        update_hook.write('git commit -a -m "`DATE` Created Automatically"' + '\n')
        update_hook.write('fi' + '\n')
except:
    pass

#####

try:
    develop_repo = git.Repo.init('./PROJECT/DEVELOP_' + project_name)
    origin = develop_repo.create_remote('origin', SERVER)
    print(develop_repo.remote('origin'))
    assert origin.exists()

    hook_develop = DEVELOP + '/.git' + '/hooks' + '/post_commit'
    with open(hook_develop, 'w') as commit_hook:
        commit_hook.write('#!/bin/sh\n')
        commit_hook.write('git log | head -1 | awk \'{print \\$2}\' > ../.develop.flag')
except Exception as inst:
    print(inst)

#####

fromDirectory = "BASE"                              ##copy from
toDirectory = "./PROJECT/DEVELOP_" + project_name   ##copy to
try:
    # shutil.copytree(fromDirectory, toDirectory, ignore=shutil.ignore_patterns('.git'))
    copy_tree(fromDirectory, toDirectory)
except Exception as inst:
    print('exception: copy')
    print(inst)
#####
try:
    release_repo = git.Repo.init('./PROJECT/RELEASE_' + project_name)
    origin = release_repo.create_remote('origin', SERVER)
    assert origin.exists()
except Exception as inst:
    print(inst)


try:
    repo = git.Repo.init('./PROJECT/')
except:
    pass

index = develop_repo.index
index.add('.')

author = git.Actor("admin", "example@mail.com")
committer = git.Actor("admin", "example@mail.com")

index.commit("hello", author=author, committer=committer)
print(develop_repo.remote('origin'))


#USER_RW = (input('Введите логин разработчика: ',))
#USER_ALL = (input('Введите логин конструктора: ',))
USER_RW = "windomainuser1name"  #RW permissions"""
USER_ALL = "windomainuser2name"  #all permissions"""


###для DEVELOP
entries = [{'AccessMode': win32security.GRANT_ACCESS,
            'AccessPermissions': 0,
            'Inheritance': win32security.CONTAINER_INHERIT_ACE |
                           win32security.OBJECT_INHERIT_ACE,
            'Trustee': {'TrusteeType': win32security.TRUSTEE_IS_USER,
                        'TrusteeForm': win32security.TRUSTEE_IS_NAME,
                        'Identifier': ''}}
            for i in range(2)]

entries[0]['AccessPermissions'] = (ntsecuritycon.GENERIC_READ | ntsecuritycon.GENERIC_WRITE)     #категории прав
entries[0]['Trustee']['Identifier'] = USER_RW                       #кому присвоить
entries[1]['AccessPermissions'] = ntsecuritycon.GENERIC_ALL
entries[1]['Trustee']['Identifier'] = USER_ALL

sd = win32security.GetNamedSecurityInfo(DEVELOP, win32security.SE_FILE_OBJECT,
        win32security.DACL_SECURITY_INFORMATION)
dacl = sd.GetSecurityDescriptorDacl()
dacl.SetEntriesInAcl(entries)
win32security.SetNamedSecurityInfo(DEVELOP, win32security.SE_FILE_OBJECT,
    win32security.DACL_SECURITY_INFORMATION |
    win32security.UNPROTECTED_DACL_SECURITY_INFORMATION,
    None, None, dacl, None)


###для RELEASE
entries = [{'AccessMode': win32security.GRANT_ACCESS,
            'AccessPermissions': 0,
            'Inheritance': win32security.CONTAINER_INHERIT_ACE |
                           win32security.OBJECT_INHERIT_ACE,
            'Trustee': {'TrusteeType': win32security.TRUSTEE_IS_USER,
                        'TrusteeForm': win32security.TRUSTEE_IS_NAME,
                        'Identifier': ''}}
            for i in range(2)]

entries[0]['AccessPermissions'] = (ntsecuritycon.GENERIC_READ)         # категории прав
entries[0]['Trustee']['Identifier'] = USER_RW                       # кому присвоить
entries[1]['AccessPermissions'] = ntsecuritycon.GENERIC_ALL         ### категории прав
entries[1]['Trustee']['Identifier'] = USER_ALL                      ### кому присвоить

sd = win32security.GetNamedSecurityInfo(RELEASE, win32security.SE_FILE_OBJECT,
        win32security.DACL_SECURITY_INFORMATION)
dacl = sd.GetSecurityDescriptorDacl()
dacl.SetEntriesInAcl(entries)
win32security.SetNamedSecurityInfo(RELEASE, win32security.SE_FILE_OBJECT,
    win32security.DACL_SECURITY_INFORMATION |
    win32security.UNPROTECTED_DACL_SECURITY_INFORMATION,
    None, None, dacl, None)


##для SERVER
entries = [{'AccessMode': win32security.GRANT_ACCESS,
            'AccessPermissions': 0,
            'Inheritance': win32security.CONTAINER_INHERIT_ACE |
                           win32security.OBJECT_INHERIT_ACE,
            'Trustee': {'TrusteeType': win32security.TRUSTEE_IS_USER,
                        'TrusteeForm': win32security.TRUSTEE_IS_NAME,
                        'Identifier': ''}}
            for i in range(2)]

"""entries[0]['AccessPermissions'] = (ntsecuritycon.GENERIC_READ)  #категории прав
entries[0]['Trustee']['Identifier'] = USER_RW      """                 #кому присвоить
entries[1]['AccessPermissions'] = ntsecuritycon.GENERIC_ALL
entries[1]['Trustee']['Identifier'] = USER_ALL

sd = win32security.GetNamedSecurityInfo(SERVER, win32security.SE_FILE_OBJECT,
        win32security.DACL_SECURITY_INFORMATION)
dacl = sd.GetSecurityDescriptorDacl()
dacl.SetEntriesInAcl(entries)
win32security.SetNamedSecurityInfo(SERVER, win32security.SE_FILE_OBJECT,
    win32security.DACL_SECURITY_INFORMATION |
    win32security.UNPROTECTED_DACL_SECURITY_INFORMATION,
    None, None, dacl, None)







"""###для файлов
entries[0]['AccessPermissions'] = (ntsecuritycon.GENERIC_READ |
                                   ntsecuritycon.GENERIC_WRITE)     #категории прав
entries[0]['Trustee']['Identifier'] = USER_RW                       #кому присвоить
entries[1]['AccessPermissions'] = ntsecuritycon.GENERIC_ALL
entries[1]['Trustee']['Identifier'] = USER_ALL

sd = win32security.GetNamedSecurityInfo(FILENAME, win32security.SE_FILE_OBJECT,
        win32security.DACL_SECURITY_INFORMATION)
dacl = sd.GetSecurityDescriptorDacl()
dacl.SetEntriesInAcl(entries)
win32security.SetNamedSecurityInfo(FILENAME, win32security.SE_FILE_OBJECT,
    win32security.DACL_SECURITY_INFORMATION |
    win32security.UNPROTECTED_DACL_SECURITY_INFORMATION,
    None, None, dacl, None)
"""



