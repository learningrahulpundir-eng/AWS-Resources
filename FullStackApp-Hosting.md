#### Full Stack App hosting on AWS (React + Node)

### Frontend
Create a simple React application, generate its production build, and upload the build files to an S3 bucket for static hosting.

### Backend
Create a Lambda function that returns employee details (currently hardcoded, but this can later be fetched from DynamoDB). Expose the Lambda through API Gateway so the frontend application can call it to retrieve employee data.

### Output
The employee list will be rendered in the UI.

## Lambda function implementation
```javascript
export const handler = async (event) => {
  const response = {
    statusCode: 200,
    headers: {
      "Access-Control-Allow-Origin": "*",
      "Access-Control-Allow-Headers": "*",
      "Access-Control-Allow-Methods": "GET,POST,OPTIONS"
    },
    body: JSON.stringify({
      message: "Employee data fetched successfully",
      employees: [
        {
          id: 1,
          name: "Rahul Sharma",
          role: "Software Engineer",
          department: "IT",
          location: "Noida"
        },
        {
          id: 2,
          name: "Priya Singh",
          role: "Data Analyst",
          department: "Analytics",
          location: "Bangalore"
        },
        {
          id: 3,
          name: "Amit Verma",
          role: "DevOps Engineer",
          department: "Cloud",
          location: "Hyderabad"
        }
      ]
    })
  };

  return response;
};
```

## Note
Enable CORS on API Gateway so the frontend application can communicate with the API Gateway.
