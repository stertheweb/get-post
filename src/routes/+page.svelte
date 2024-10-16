<script>

const USER_KEY = "8fddd0c491b1b7809ba31d33b56c"
const GET_EXAMPLE_ENDPOINT = `https://candidate.hubteam.com/candidateTest/v3/problem/test-dataset-answer?userKey=${USER_KEY}`
const GET_ENDPOINT = `https://candidate.hubteam.com/candidateTest/v3/problem/dataset?userKey=${USER_KEY}`
const POST_ENDPOINT = `https://candidate.hubteam.com/candidateTest/v3/problem/test-result?userKey=${USER_KEY}`

async function fetchData() {
	const response = await fetch(GET_ENDPOINT)
    return response.json()
}

async function postData(data) {
  const response = await fetch(POST_ENDPOINT, {
    method: "POST",
    body: JSON.stringify(data),
    headers: {
      "Content-type": "application/json; charset=UTF-8",
    },
  })
  return response.json()
}

function convertTimestampToDate(timestamp) {
  const date = new Date(timestamp * 1000); // Convert to milliseconds
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0'); // Add 1 because months are 0-indexed
  const day = String(date.getDate()).padStart(2, '0');

  return `${year}-${month}-${day}`;
}

function filter(data) {
	const result = [];

    const calls = data.callRecords;

    // Group by date and customerId
    const groupedCalls = {};

    calls.forEach(call => {
        const callDate = convertTimestampToDate(call.startTimestamp);
        const customerId = call.customerId;

        const key = `${customerId}-${callDate}`;
        
        if (!groupedCalls[key]) {
            groupedCalls[key] = [];
        }

        groupedCalls[key].push(call);
    });

    // Calculate concurrent calls per date and customer
    for (const key in groupedCalls) {
        const [customerId, date] = key.split("-");
        const customerCalls = groupedCalls[key];
        
        customerCalls.forEach((currentCall, index) => {
            const concurrentCalls = customerCalls.filter(call => 
                (call.startTimestamp <= currentCall.endTimestamp && call.endTimestamp >= currentCall.startTimestamp)
            );

            const concurrentCallIds = concurrentCalls.map(call => call.callId);

            result.push({
                date: convertTimestampToDate(currentCall.startTimestamp),
                customerId: parseInt(customerId),
                maxConcurrentCalls: concurrentCalls.length,
                callIds: concurrentCallIds,
                timestamp: currentCall.endTimestamp
            });
        });
    }

    return result;
}

function start() {
        (
            async()=>{
                let callsData = await fetchData()
				// console.log(callsData)
                let filteredData = filter(callsData)
				console.log("filteredData: ", filteredData)
                let postResponse = await postData(filteredData)
                console.log("post: ", postResponse)
            }
        )()
}

function app(){
    start()
}

app()
</script>

<svelte:head>
	<title>Home</title>
	<meta name="description" content="Svelte demo app" />
</svelte:head>

<section>

</section>
