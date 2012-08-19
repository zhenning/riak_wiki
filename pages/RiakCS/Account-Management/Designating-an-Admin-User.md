<div class="info"><div class="title">Riak CS Only</div>This documentation applies only to Riak Cloud Storage, a commercial extension to <a href="http://wiki.basho.com/Riak.html">Riak</a>. To talk to us about using Riak CS, <a href="http://info.basho.com/Wiki_Contact_RiakCS.html" target="_blank">let us know</a>.</div>

# Designating an Admin User
Once a user has been created, you should designate a user as an admin by editing the replacing the `admin_key` and `admin_secret` in `app.config` with the user's credentials. Once this is done, do not forget to update the same credentials in the Stanchion `app.config` as well.

<div class="note"><div class="title">Note</div>
This is a powerful role and gives the designee an administrative capabilities within the system. As such caution should be used to protect the access credentials of the admin user.
</div>