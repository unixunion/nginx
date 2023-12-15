# NGINX JA4 Module

The module source [source](https://bitbucket.kindredgroup.com/bitbucket/projects/CDS/repos/nginx-dd/browse/nginx-centos7/src/nginx-mod_ja4), it was taken from [darksail-mod](https://github.com/darksail-ai/nginx/tree/darksail-mod/module), it has not been modified at time of writing this. 15 Dec 2023

There is also this [repo](https://bitbucket.kindredgroup.com/bitbucket/projects/CDS/repos/nginx-mod_ja4/browse), though its not integrated into nginx-dd build yet.

The module is added to nginx at compile time via `-add-module` configure argument, as can be seen in [build.sh](https://bitbucket.kindredgroup.com/bitbucket/projects/CDS/repos/nginx-dd/browse/nginx-centos7/scripts/build.sh#129-162)


# Patching NGINX to use JA4 Module

We have this NGINX for which contains branches `ja4-X.Y.Z` which is a specific patched version of nginx. The original patch is from darksail-ai, but its been modified to fix a [deathloop](https://github.com/darksail-ai/nginx/blob/darksail-mod/src/event/ngx_event_openssl.c#L1906-L1920): 

Generally patches are long lived, but if there are changes in the same areas as the JA4 patch touches in vanilla nginx, then you would see chunks failing to apply. And would need to re-integrate these.

## Forward Porting

When we have a new version of NGINX, the patch should work, as its not so intrusive, but if there are issues in patch chunks, forward porting might
be required.

Here is a overview of how to achieve this.

```bash
export NGINX_CURRENT_VERSION=1.21.4
export NGINX_UPGRADE_VERSION=1.25.3
git clone https://bitbucket.kindredgroup.com/bitbucket/scm/cds/nginx.git
cd nginx
git checkout ja4-${NGINX_CURRENT_VERSION}
git remote add upstream https://github.com/nginx/nginx.git
git fetch upstream --tags
git pull upstream release-${NGINX_UPGRADE_VERSION}
git add src/event/ngx_event_openssl.c                                                                                                  git:ja4-1.21.4*
git add src/event/ngx_event_openssl.h                                                                                                  git:ja4-1.21.4*
git add src/http/modules/ngx_http_ssl_module.c 
```

Now resolve any issues in the patching if required.

### Generate the Patch

Generate a patch, review and adapt as needed. 

```bash
git diff release-${NGINX_UPGRADE_VERSION} ':(exclude)README.md' | tee ja4.patch
```

### Using the Patch

The patch goes into the [nginx-dd](https://bitbucket.kindredgroup.com/bitbucket/projects/CDS/repos/nginx-dd/browse/nginx-centos7/src/nginx-mod_ja4), also this is where the module lives.

