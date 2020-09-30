Deploy the static website to S3/CloudFront
======================================

.. image:: https://travis-ci.org/jonls/s3-deploy-website.svg?branch=master
   :alt: Build Status
   :target: https://travis-ci.org/jonls/s3-deploy-website

.. image:: https://badge.fury.io/py/s3-deploy-website.svg
   :alt: PyPI badge
   :target: http://badge.fury.io/py/s3-deploy-website

.. image:: https://coveralls.io/repos/jonls/s3-deploy-website/badge.svg?branch=master&service=github
   :alt: Test coverage
   :target: https://coveralls.io/github/jonls/s3-deploy-website?branch=master

This is a deployment tool for uploading static websites to S3. If CloudFront is
used for hosting the website, the uploaded files can be automatically
invalidated in the CloudFront distribution. A prefix tree is used to
minimize the number of invalidations since only a limited number of free
invalidations are available per month.

The configuration is stored in a YAML file like this:

.. code-block:: yaml

    site: _site
    s3_bucket: example.com
    cloudfront_distribution_id: XXXXXXXXXXX

    cache_rules:
      - match: "/assets/*"
        maxage: 30 days

      - match_regexp: '^assets/image-\d{3}-.*\.png$'
        maxage: 90 days

      - match: "/css/*"
        maxage: 30 days

      - match: "*"
        maxage: 1 hour

The ``site`` is the directory of the static website relative to the location
of the configuration file. For example, Jekyll will generate the static site
in the ``_site`` directory as specified above. If you save the configuration
file as ``.s3_website.yaml`` you can simply run ``s3-deploy-website`` from the
same directory:

.. code-block:: shell

    $ cd jekyll-site/
    $ ls .s3_website.yaml
    .s3_website.yaml
    $ s3-deploy-website

Credentials
-----------

AWS credentials can be provided through the environment variables
``AWS_ACCESS_KEY_ID`` and ``AWS_SECRET_ACCESS_KEY``.

.. code-block:: shell

    $ export AWS_ACCESS_KEY_ID=XXXXXX
    $ export AWS_SECRET_ACCESS_KEY=XXXXXX
    $ s3-deploy-website

They can also be provided through the various configuration files that boto_
reads.

.. _boto: https://boto3.readthedocs.io/en/latest/guide/configuration.html

Configuration file
------------------

**site**
    The directory of the static content to be uploaded (relative to
    the location of the configuration file (e.g. ``_site`` for Jekyll sites).

**s3_bucket**
    The name of the S3 bucket to upload the files to. You have to allow the
    actions ``s3:GetObject``, ``s3:PutObject``, ``s3:DeleteObject`` and
    ``s3:ListBucket`` on the bucket and the keys e.g.
    ``arn:aws:s3:::example.com`` and ``arn:aws:s3:::example.com/*``.

**s3_reduced_redundancy**
    An optional boolean to indicate whether the files should be uploaded
    to `reduced redundancy`_ storage.

**cloudfront_distribution_id**
    The CloudFront distribution to invalidate after uploading new files. Only
    files that were changed will be invalidated. You have to allow the
    action ``CloudFront: CreateInvalidation``.

**cache_rules**
    A list of rules to determine the cache configuration of the uploaded files.
    The ``match`` key specifies a pattern that the rule applies to. This uses
    glob-style matching (with ``*`` and ``?``). Matching can also be performed
    with regular expressions by using ``match_regexp``. Only the
    first rule to match a given key will be used. The ``maxage`` key
    specifies the time to cache the file. The value should be either a number
    of seconds or a string like ``30 days``, ``5 minutes, 30 seconds``, etc.

**endpoint_url**
    (Optional) For setting custom endpoint for boto3:
    `Boto3 Session reference`_

.. _`reduced redundancy`: https://aws.amazon.com/s3/reduced-redundancy/
.. _`Boto3 Session reference`: https://boto3.amazonaws.com/v1/documentation/api/latest/reference/core/session.html#boto3.session.Session.resource

Similar software
----------------

The configuration in ``.s3_website.yaml`` was inspired by s3_website_ although
the options supported by s3_website are slightly different.

.. _s3_website: https://github.com/laurilehmijoki/s3_website

License
-------

MIT.
