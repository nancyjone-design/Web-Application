<form action="{% url "myapp:upload_csv" %}" method="POST" enctype="multipart/form-data" class="form-horizontal"> 
{% csrf_token %}
<div class="form-group">
    <label for="name" class="col-md-3 col-sm-3 col-xs-12 control-label">File: </label>
    <div class="col-md-8">
        <input type="file" name="csv_file" id="csv_file" required="True" class="form-control">
    </div>                    
</div>
<div class="form-group">                    
    <div class="col-md-3 col-sm-3 col-xs-12 col-md-offset-3" style="margin-bottom:10px;">
         <button class="btn btn-primary"> <span class="glyphicon glyphicon-upload" style="margin-right:5px;"></span>Upload </button>
    </div> 
</div>
</form>
def upload_csv(request):
	data = {}
	if "GET" == request.method:
		return render(request, "myapp/upload_csv.html", data)
    # if not GET, then proceed
	try:
		csv_file = request.FILES["csv_file"]
		if not csv_file.name.endswith('.csv'):
			messages.error(request,'File is not CSV type')
			return HttpResponseRedirect(reverse("myapp:upload_csv"))
      if csv_file.multiple_chunks():
			messages.error(request,"Uploaded file is too big (%.2f MB)." % (csv_file.size/(1000*1000),))
			return HttpResponseRedirect(reverse("myapp:upload_csv"))

		file_data = csv_file.read().decode("utf-8")		

		lines = file_data.split("\n")
		for line in lines:						
			fields = line.split(",")
			data_dict = {}
			data_dict["name"] = fields[0]
			data_dict["start_date_time"] = fields[1]
			data_dict["end_date_time"] = fields[2]
			data_dict["notes"] = fields[3]
			try:
				form = EventsForm(data_dict)
				if form.is_valid():
					form.save()					
				else:
					logging.getLogger("error_logger").error(form.errors.as_json())												
			except Exception as e:
				logging.getLogger("error_logger").error(repr(e))					
				pass

	except Exception as e:
		logging.getLogger("error_logger").error("Unable to upload file. "+repr(e))
		messages.error(request,"Unable to upload file. "+repr(e))

	return HttpResponseRedirect(reverse("myapp:upload_csv"))
