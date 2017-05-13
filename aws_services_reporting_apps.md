# AWS Services for Reporting Application

One can simply use AWS Elastic Beanstalk (EB) to quickly deploy a web application. Following approach was used to create and deploy reporting ap using AWS EB:

* Create Reporting application using Springboot; Expose the REST APIs using @RestController 
* Dockerize the reporting app
* Create an application on AWS EB which is based on [Single Container Docker platform configuration](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/create_deploy_docker.html).
* Create a Dockerrun.aws.json specifying the container details. Following is the sample code:
```
{
  "AWSEBDockerrunVersion": "1",
  "Image": {
    "Name": "ajitesh/srapis",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": "8080"
    }
  ],
  "Logging": "/var/log"
}
```
* Deploy the app manually using AWS EB application console or using Jenkins using instructions mentioned on the following page: [Continuous Deplivery of Reporting Apps to AWS](https://github.com/eajitesh/Reporting-Solution-on-AWS/blob/master/cicd_reporting_apps_aws.md)

One could as well use multi-container docker platform configuration of AWS EB or [AWS ECS](http://docs.aws.amazon.com/AmazonECS/latest/developerguide/Welcome.html) to deploy the reporting app/APIs.

## Reporting App using Springboot - Key Considerations

Following are some of the key highlights to be considered when building reporting apps which interacts with AWS ElasticSearch as data storage.

* Springboot Main Class Configuration can be done in following way when using Spring Data to avoid errors related Hibernate and ElasticSearch configurations. Note the classes excluded in @SpringBootApplication annotation.

```
@SpringBootApplication(exclude = { ElasticsearchAutoConfiguration.class, ElasticsearchDataAutoConfiguration.class,
		DataSourceAutoConfiguration.class, HibernateJpaAutoConfiguration.class })
public class ReportingApIsApplication {

	public static void main(String[] args) {
		SpringApplication.run(ReportingApIsApplication.class, args);
	}
}
```

* Create Endpoint to expose REST API for querying the reporting data. Following is the code sample:

```
@RestController
public class ReportingEndpoint {
	
	private static final Logger log = LoggerFactory.getLogger(ReportingEndpoint.class);
	
	private CustomerAppointmentsService customerAppointmentsService;
	
	@Autowired
	public ReportingEndpoint(CustomerAppointmentsService customerAppointmentsService) {
		this.customerAppointmentsService = customerAppointmentsService;
	}
	
	@PostMapping(value="/customer_appointments_statuses", produces="application/json")
	public @ResponseBody CustomerAppointmentsStatusesResponse getCustomerAppointmentsStatusesyDate(ModelMap model, @RequestBody CustomerAppointmentsStatusesRequest customerAppointmentsRequest) {
		List<CustomerAppointments> customerAppointments;
		CustomerAppointmentsStatusesResponse casr = new CustomerAppointmentsStatusesResponse();
	        customerAppointments = customerAppointmentsService.findByAppointmentDateRange(customerAppointmentsRequest.getFromDate(), customerAppointmentsRequest.getToDate());
		...
		...
		return casr;
    }	

}
```

* Create Service class to interact with data access object. Following is the code sample:

```
@Component
public class CustomerAppointmentsServiceImpl implements CustomerAppointmentsService {

	private static final Logger log = LoggerFactory.getLogger(CustomerAppointmentsServiceImpl.class);

	private CustomerAppointmentsDAO customerAppointmentsDAO;
	
	@Autowired
	public CustomerAppointmentsServiceImpl(CustomerAppointmentsDAO customerAppointmentsDAO) {
		this.customerAppointmentsDAO = customerAppointmentsDAO;
	}
	
	@Override
	public List<CustomerAppointments> findByAppointmentDateRange(String fromDateStr, String toDateStr) throws IncorrectDateFormatException {
		Date fromDate = null;
		Date toDate = null;
		try {
			fromDate = DateUtils.dateFromString(fromDateStr);
			toDate = DateUtils.dateFromString(toDateStr);
		} catch (ParseException e) {
			log.error(e.getMessage(), e);
		}
		if(fromDate == null || toDate == null) {
			throw new IncorrectDateFormatException("Date format seems to be incorrect. Please check and truy again");
		}
		return customerAppointmentsDAO.findByAppointmentDateRange(fromDate.getTime(), toDate.getTime());
	}
}
```

* Usage of Spring Data for interacting with AWS ES. Following is the sample DAO file which queries customer appointments based on date range

```
public interface CustomerAppointmentsDAO
		extends ElasticsearchRepository<CustomerAppointments, String> {
	@Query("{\"query\": {\"range\" : {\"appointmentDate\": {\"gt\":\"?0\", \"lt\": \"?1\"}}}}")
	List<CustomerAppointments> findByAppointmentDateRange(long fromDate, long toDate);
}
```

