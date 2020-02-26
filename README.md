1. Activate the maintenance mode:
heroku maintenance:on -a <app_name>

2. To export the data from your Heroku Postgres database, create a new backup and download it.

heroku pg:backups:capture -a <app_name>
heroku pg:backups:download -a <app_name>

3. Create database on Amazon RDS
- check access database: psql -U $RDS_ROOT_USER -h $NAME.$ID.$DATACENTER.rds.amazonaws.com --dbname=$DATABASE_NAME

4. Add the RDS certificate to your application
In order to let your application talk to RDS, you have to use a SSL certificate:
cd /path/to/your/app
curl https://s3.amazonaws.com/rds-downloads/rds-combined-ca-bundle.pem > ./config/rds-combined-ca-bundle.pem
git add config/rds-combined-ca-bundle.pem
git commit -m "Add Amazon SSL certificate"
git push heroku

5. Load the dump on RDS
Use pg_restore to load your data in your new database: pg_restore --verbose --clean --no-acl --no-owner -h $NAME.$ID.$DATACENTER.rds.amazonaws.com -U $RDS_ROOT_USER -d $DATABASE_NAME /tmp/latest.dump

6. Switch the app to RDS
So, assuming you use the heroku-postgresql addon:
heroku addons:destroy heroku-postgresql -a <app_name>

Then set the DATABASE_URL environment variable:
heroku config:set DATABASE_URL="postgres://$RDS_ROOT_USER:$PASSWORD@$NAME.$ID.$DATACENTER.rds.amazonaws.com/$DATABASE_NAME?sslca=config/rds-combined-ca-bundle.pem" -a <app_name>

7. Deactivate the maintenance mode:
heroku maintenance:off -a <app_name>
