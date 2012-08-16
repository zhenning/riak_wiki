# Designating an Admin User
Once a user has been created, you should designate a user as an admin by replacing the `admin_key` and `admin_secret` in `etc/app.config` with the user's credentials. Once done, do not forget to update the same credentials in `stanchion`.

<div class="note"><div class="title">Note</div>
This is a powerful role and gives the designee administrative capabilities within the system. As such caution should be used to protect the access credentials of the admin user.
</div>