## Week 1 Homework

In this homework we'll prepare the environment 
and practice with terraform and SQL

## Question 1. Google Cloud SDK

Install Google Cloud SDK. What's the version you have? 

To get the version, run `gcloud --version`

`369.0.0`

## Google Cloud account 

Create an account in Google Cloud and create a project.


## Question 2. Terraform 

Now install terraform and go to the terraform directory (`week_1_basics_n_setup/1_terraform_gcp/terraform`)

After that, run

* `terraform init`
* `terraform plan`
* `terraform apply` 

Apply the plan and copy the output (after running `apply`) to the form.

It should be the entire output - from the moment you typed `terraform init` to the very end.

```bash
Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.

var.project
  Your GCP Project ID

  Enter a value: dtc-de-course-339308


Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the
following symbols:
  + create

Terraform will perform the following actions:

  
# google_bigquery_dataset.dataset will be created
  + resource "google_bigquery_dataset" "dataset" {
      + creation_time              = (known after apply)
      + dataset_id                 = "trips_data_all"
      + delete_contents_on_destroy = false
      + etag                       = (known after apply)
      + id                         = (known after apply)
      + last_modified_time         = (known after apply)
      + location                   = "europe-west6"
      + project                    = "dtc-de-course-339308"
      + self_link                  = (known after apply)

      + access {
          + domain         = (known after apply)
          + group_by_email = (known after apply)
          + role           = (known after apply)
          + special_group  = (known after apply)
          + user_by_email  = (known after apply)

          + view {
              + dataset_id = (known after apply)
              + project_id = (known after apply)
              + table_id   = (known after apply)
            }
        }
    }

  # google_storage_bucket.data-lake-bucket will be created
  + resource "google_storage_bucket" "data-lake-bucket" {
      + force_destroy               = true
      + id                          = (known after apply)
      + location                    = "EUROPE-WEST6"
      + name                        = "dtc_data_lake_dtc-de-course-339308"
      + project                     = (known after apply)
      + self_link                   = (known after apply)
      + storage_class               = "STANDARD"
      + uniform_bucket_level_access = true
      + url                         = (known after apply)

      + lifecycle_rule {
          + action {
              + type = "Delete"
            }

          + condition {
              + age                   = 30
              + matches_storage_class = []
              + with_state            = (known after apply)
            }
        }

      + versioning {
          + enabled = true
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

google_bigquery_dataset.dataset: Creating...
google_storage_bucket.data-lake-bucket: Creating...
google_bigquery_dataset.dataset: Creation complete after 4s [id=projects/dtc-de-course-339308/datasets/trips_data_all]
google_storage_bucket.data-lake-bucket: Creation complete after 5s [id=dtc_data_lake_dtc-de-course-339308]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

## Prepare Postgres 

Run Postgres and load data as shown in the videos

We'll use the yellow taxi trips from January 2021:

```bash
wget https://s3.amazonaws.com/nyc-tlc/trip+data/yellow_tripdata_2021-01.csv
```

You will also need the dataset with zones:

```bash 
wget https://s3.amazonaws.com/nyc-tlc/misc/taxi+_zone_lookup.csv
```

Download this data and put it to Postgres

## Question 3. Count records 

How many taxi trips were there on January 15?

Consider only trips that started on January 15.

```bash
select count(index) from yellow_taxi_data
where date(tpep_pickup_datetime) = '2021-01-15'
```


## Question 4. Largest tip for each day

Find the largest tip for each day. 
On which day it was the largest tip in January?

Use the pick up time for your calculations.

(note: it's not a typo, it's "tip", not "trip")

```bash
select date(tpep_pickup_datetime) from yellow_taxi_data
where tip_amount = (select max(tip_amount) from yellow_taxi_data)
```


## Question 5. Most popular destination

What was the most popular destination for passengers picked up 
in central park on January 14?

Use the pick up time for your calculations.

Enter the zone name (not id). If the zone name is unknown (missing), write "Unknown" 

```bash
with a as(
SELECT
t."index" id,
tpep_pickup_datetime,
tpep_dropoff_datetime,
total_amount,
zpu."Zone" pickup_zone,
zdo."Zone" dropout_zone
FROM YELLOW_TAXI_DATA t join zones zpu
	on t."PULocationID" = zpu."LocationID"
	join zones zdo
	on t."DOLocationID" = zdo."LocationID"
where DATE(tpep_pickup_datetime) = '2021-01-14'),
b as (select dropout_zone, count(dropout_zone) total from a
where pickup_zone = 'Central Park'
group by 1)
select dropout_zone, total from b
where total = (select max(total) from b)
```

## Question 6. Most expensive locations

What's the pickup-dropoff pair with the largest 
average price for a ride (calculated based on `total_amount`)?

Enter two zone names separated by a slash

For example:

"Jamaica Bay / Clinton East"

If any of the zone names are unknown (missing), write "Unknown". For example, "Unknown / Clinton East". 

```bash
with a as(
SELECT
t."index" id,
tpep_pickup_datetime,
tpep_dropoff_datetime,
abs(total_amount) as total_amount,
COALESCE(zpu."Zone",'Unknown') AS pickup,
COALESCE(zdo."Zone",'Unknown') AS dropout
FROM YELLOW_TAXI_DATA t join zones zpu
	on t."PULocationID" = zpu."LocationID"
	join zones zdo
	on t."DOLocationID" = zdo."LocationID"),
b as(
select 
	CONCAT(pickup, ' / ', dropout) AS combined,
	total_amount
from a
),
c as(
select combined, avg(total_amount) averagefee from b
group by 1)
select combined from c
where averagefee = (select max(averagefee) from c)
```

## Submitting the solutions

* Form for submitting: https://forms.gle/yGQrkgRdVbiFs8Vd7
* You can submit your homework multiple times. In this case, only the last submission will be used. 

Deadline: 26 January (Wednesday), 22:00 CET

