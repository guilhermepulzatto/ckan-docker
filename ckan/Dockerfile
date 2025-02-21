# (C) Copyright Promptly Health 2025.
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM ckan/ckan-base:2.11

ARG build_date
ARG vcs_ref
ARG VERSAO=1.0.0
ARG BOM_PATH="/docker/promptly"
ARG APP_DIR="/srv/app"
ARG SRC_DIR="/srv/app/src"

SHELL ["/bin/bash", "-o", "pipefail", "-c"]

LABEL org.label-schema.build-date=$build_date \
      org.label-schema.name="ckan" \
      org.label-schema.description="CKAN image with custom extensions"

USER root

# # Install any extensions needed by your CKAN instance
# # - Make sure to add the plugins to CKAN__PLUGINS in the .env file
# # - Also make sure all provide all extra configuration options, either by:
# #   * Adding them to the .env file (check the ckanext-envvars syntax for env vars), or
# #   * Adding extra configuration scripts to /docker-entrypoint.d folder) to update
# #      the CKAN config file (ckan.ini) with the `ckan config-tool` command
# #
# # See README > Extending the base images for more details
# #
# # For instance:
# #

# Update pip and packages - alipine
# RUN pip install --no-cache-dir --upgrade pip && \
#     apk update && apk upgrade && \
#     apk add --no-cache --virtual .build-deps gcc musl-dev libffi-dev openssl-dev python3-dev

# Update pip and packages - ubuntu
RUN pip install --no-cache-dir --upgrade pip && \
    apt-get update && apt-get upgrade -y && \
    apt-get install -y --no-install-recommends gcc musl-dev libffi-dev libssl-dev python3-dev && \
    pip install pyld


## Harvester - DCAT uses CKAN-Harvester ###
RUN pip3 install -e 'git+https://github.com/ckan/ckanext-harvest.git@master#egg=ckanext-harvest' && \
    pip3 install -r ${APP_DIR}/src/ckanext-harvest/pip-requirements.txt
#will also require gather_consumer and fetch_consumer processes running (please see https://github.com/ckan/ckanext-harvest)

## DCAT ###
RUN pip install -e git+https://github.com/ckan/ckanext-dcat.git#egg=ckanext-dcat
RUN pip install -r https://raw.githubusercontent.com/ckan/ckanext-dcat/v2.2.0/requirements.txt

## DCAT Pre-built schemas###
RUN pip install -e "git+https://github.com/ckan/ckanext-scheming.git#egg=ckanext-scheming"

## Keycloak ###
RUN pip3 install -e git+https://github.com/keitaroinc/ckanext-keycloak.git#egg=ckanext-keycloak \
    && pip3 install -r /srv/app/src/ckanext-keycloak/requirements.txt

# Copy custom initialization scripts
COPY --chown=ckan-sys:ckan-sys docker-entrypoint.d/* /docker-entrypoint.d/

# Apply any patches needed to CKAN core or any of the built extensions (not the
# runtime mounted ones)
COPY --chown=ckan-sys:ckan-sys patches ${APP_DIR}/patches

USER ckan

RUN for d in $APP_DIR/patches/*; do \
        if [ -d $d ]; then \
            for f in `ls $d/*.patch | sort -g`; do \
                cd $SRC_DIR/`basename "$d"` && echo "$0: Applying patch $f to $SRC_DIR/`basename $d`"; patch -p1 < "$f" ; \
            done ; \
        fi ; \
    done
