py-minisign
===========

Missing python `minisign <https://github.com/jedisct1/minisign>`_ library.

This is a private fork with the following changes:
 * signature length should be fully deterministic
 * replace time.time with custom datetime string of deterministic length (14) that ignores milliseconds
 * Remove untrusted comments from Signature class completely
 * Trusted comment of Signature class replaced by "Subject"
 * Subject is now first field of Signature
 * Subject is now of fixed format {blake2b(input):{deterministic_datetime}}, this is designed for the use case of addressing a signed package to a particular node. for example: sk.sign(subject="recipient") will create the subject "hashed_recipient:deterministic_datetime".  Theres no security it just lets the client check if its addressed to them, and if it is newer.  This subject IS validated per upstream's "trusted_comment"
 * Remove prefixes from comment and subject
 * restructure signature drop and load to work with our new signature structure


Library
-------

.. code:: python

    import os
    import minisign

    # verify

    pk = minisign.PublicKey.from_base64(
        'RWQf6LRCGA9i53mlYecO4IzT51TGPpvWucNSCh1CBM0QTaLn73Y7GFO3')
    sig = minisign.Signature.from_bytes(
        b'untrusted comment: signature from minisign secret key\n'
        b'RWQf6LRCGA9i59SLOFxz6NxvASXDJeRtuZykwQepbDEGt87ig1BNpWaVWuNrm73YiIiJbq71Wi+dP9eKL8OC351vwIasSSbXxwA=\n'
        b'trusted comment: timestamp:1555779966\tfile:test\n'
        b'QtKMXWyYcwdpZAlPF7tE2ENJkRd1ujvKjlj1m9RtHTBnZPa5WKU5uWRs5GoP5M/VqE81QFuMKI5k/SfNQUaOAA=='
    )
    pk.verify(b'test', sig)

    # sign

    sk = minisign.SecretKey.from_file('/path/to/secret.key')
    sk.decrypt('strong_password')
    sig = sk.sign(b'very important data')

    # generate key pair

    key_pair = minisign.KeyPair.generate()
    sk = key_pair.secret_key
    pk = key_pair.public_key

    # save key

    sk.encrypt('strong_password')
    with open(os.open('/path/to/secret.key', os.O_CREAT | os.O_WRONLY, 0o600), 'wb') as f:
        f.write(bytes(sk))
