There are 3 ways to call the API.  The higher level request used, the more accurate the pre-qualification details will be.  All responses will be in JSON.

## Production endpoint
https://www.creditkey.com/app/prequalification

## Staging endpoint
https://staging.creditkey.com/app/prequalification


# Basic Request:

```
curl -d "email=yo@com.com&company_name=CompanyName&gross_annual_revenue=49999&owner_first_name=William&owner_last_name=McWilliam&street=123&city=Cityplace&state=CA&zip=12345&public_key=XXXXXXXX&shared_secret=XXXXXXXX" -X POST https://www.creditkey.com/app/prequalification
```

Fields:

* email
* company_name
* gross_annual_revenue (integer in dollars)
* owner_first_name
* owner_last_name
* street
* city
* state (postal abbrev.)
* zip

# Intermediate Request:

```
curl -d "fico_scale=4&email=yo@com.com&company_name=CompanyName&gross_annual_revenue=49999&owner_first_name=William&owner_last_name=McWilliam&street=123&city=Cityplace&state=CA&zip=12345&public_key=XXXXXXXX&shared_secret=XXXXXXXX" -X POST https://www.creditkey.com/app/prequalification
```

Additional fields:

* fico_scale (integer, enumerated)

fico_scale values:

* 1 (Poor)
* 2 (Fair)
* 3 (Good)
* 4 (Very good)
* 5 (Excellent)

# Advanced Request

```
curl -d "fico_score=705&email=yo@com.com&company_name=CompanyName&gross_annual_revenue=49999&owner_first_name=William&owner_last_name=McWilliam&street=123&city=Cityplace&state=CA&zip=12345&public_key=XXXXXXXX&shared_secret=XXXXXXXX" -X POST https://www.creditkey.com/app/prequalification
```

Additional fields:

* fico_score (integer, value between 300-850)


# Possible Responses

Positive Response

```
{:status=>:pass, :msg=>"CompanyName is pre-approved for up to $6,000.00 with Credit Key. 0% for 30 days, as low as 1% per month for longer terms. Pay over time with easy monthly terms, no hidden fees. Checking your rate will not affect your credit. <a href=\"https://www.creditkey.com/app/apply/{merchant-slug}\">Click to Apply Now</a>", :estimated_tcl=>6000, :estimated_rate=>1}
```

Negative Response - fico failure
```
{:status=>:fail, :msg=>"CompanyName is unlikely to qualify for Credit Key terms. Minimum FICO score is 650. "}
```

Negative Response - GAR failure
```
{:status=>:fail, :msg=>"CompanyName is unlikely to qualify for Credit Key terms. Minimum annual revenue is $40,000. "}
```

Negative Response - GAR + Fico failure
```
{:status=>:fail, :msg=>"CompanyName is unlikely to qualify for Credit Key terms. Minimum FICO score is 650. Minimum annual revenue is $40,000. "}
```

