For Phase 1 :
I learned what are the cases to think when handling with a distributed independent system
I built this phase with 2 systems in mind 
	- Minio for actual file storage
	- Postgres for Metadata storage for querying better
	- So first all files go through the postgres and gets saved its various metadata and the file is uploaded by the **backend** 
In these process many things can go wrong 
	- what if the Metadata upload fails - > easy we just revert back to user saying file upload failed 
	- what if the Metadata succeeds but minio fails -> we need a way to go back to a consistent state
	- So we need retries to try uploading before letting user handle it again 
	- We cant just let all errors to propage a retries , what if some errors like bucket-name validity etc,. can never be able to upload , 
	- So we need to separate the errors with proper distinction for which can be retried , for example network errors. timeout errors , etc.
- So in this phase I learned how to handle inconsistencies and retries. 

For Phase 2:
We can see a massive bottleneck for the server, rightnow the server has to do all the uploads and handle sending the files to minio for ALL THE USERS WE COULD HAVE. which is massive hit on our servers , instead if there were a way to let the users browser handle the upload where we handle the structure and management ,
Here is where the Presigning URLs come into picture. 
with this approach the minio client will returned a temporary signed url which the browser can use to upload files to minio directly 

For Phase 3
Alright now that the user is uploading the files for us , we dont need to worry about that 
but there is a small problem , when user decides to upload a large , i mean a huge file. it takes a very long time , with time comes uncertainty so if the user uploads a 5gb file in the near end the upload fails due to network error, there is no way to recover from this  the user has to upload the file from first , whomp whomp !! its very sad and frustrating. 
so we make the upload small parts and resumable so the user can worry less about uncertainty.
so minio provides the feature to create a multipart upload 
	- minio provides a upload id , in the server side we split the file as parts and send the client the part number and the presigned url for that part 
	- the client takes these information splits the file accordingly to that part and sends that part of the file to that part number 
		- for example the parts - 100
		- part number the client have is 2
		- part size is 10mb
		- then the user fetchs 10mb to 20mb of the file
		- then uploads that chunk to that presiged url
	- after every upload the client makes note of the part number and etag of the uploaded part and sends all the things back to server 
	- the server completes the upload and merges all the file for them 
We can also upload these part parallelly , 

Resuming:
in case of any failure , we can store locally in client about the last upload id , file name and size , ask user to upload the same file again and now with this we can ask the backend what parts were uploaded success fully , then only upload the parts that are not uploaded then ask the server to complete the upload 


Phase 4:
is to using message brokers to let our application pass needed data to the queues and other background services can take this 
much like celery , (celery also uses brokers inside to communicate with workers)
all basics of RabbitMQ

