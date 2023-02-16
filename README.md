# OpenCL:

1. recuperer la chaine de caractere qui contient le kernel dans un fichier
2. creer les buffers qui serviront d'arguments d'entree a la fonction et les remplir de donnees
3. recuperer les infos de plateforme avec clGetPlatformIDs et recuperer les infos des device avec clGetDeviceIDs\
`clGetPlatformIDs(1, &platform_id, &ret_num_platforms);`\
pourquoi 1 en premier argument ?\
>`cl_int clGetPlatformIDs(cl_uint num_entries, cl_platform_id *platforms,cl_uint *num_platforms);`
>num_entries\
>The number of cl_platform_id entries that can be added to platforms. If platforms is not NULL, the num_entries must be greater than zero.

4. creer un context avec clCreateContext\
` cl_context context = clCreateContext( NULL, ret_num_devices, &device_id, NULL, NULL, &ret);`

5. ceration d'une file avec clCreateCommandQueue dans laquelle on renseignera le contexte et l'id sur laquelle on veut mettre la file\
`cl_command_queue command_queue = clCreateCommandQueue(context, device_id, 0, &ret);`

6. creation des buffers cote device avec clCreateBuffer, donc un buffer par parametre de kernel (entrees/sorties)\
`cl_mem a_mem_obj = clCreateBuffer(context, CL_MEM_READ_ONLY, LIST_SIZE * sizeof(int), NULL, &ret);`

7. on remplit les buffer cote device avec les buffer cote host avec la fonction clEnqueueWriteBuffer, donc on ne remplit que les buffer d'entrees\
`ret = clEnqueueWriteBuffer(command_queue, a_mem_obj, CL_TRUE, 0,LIST_SIZE * sizeof(int), A, 0, NULL, NULL);`

8. on cree un programme avec la fonction clCreateProgram qui va servir a recuperer le code dans la chaine de caracteres de l'etape 1\
`cl_program program = clCreateProgramWithSource(context, 1,(const char **)&source_str, (const size_t *)&source_size, &ret);`

9. On "compile" le kernel avec la fonction clBuildProgram\
`ret = clBuildProgram(program, 1, &device_id, NULL, NULL, NULL);`

10. On cree le kernel avec la fonction clCreateKernel\
`cl_kernel kernel = clCreateKernel(program, "vector_add", &ret);`

11. On "precharge" les arguments d'entree du kernel avec la fonction clSetKernelArg\
`ret = clSetKernelArg(kernel, 0, sizeof(cl_mem), (void *)&a_mem_obj);`

12. Execution du kernel sur le device OpenCL  avec la fonction clEnqueueNDRangeKernel\
`ret = clEnqueueNDRangeKernel(command_queue, kernel, 1, NULL, &global_item_size, &local_item_size, 0, NULL, NULL);`
>work_dim ?\
>`cl_int clEnqueueNDRangeKernel (cl_command_queue command_queue,cl_kernel kernel,cl_uint work_dim,const size_t *global_work_offset,const size_t *global_work_size,const size_t *local_work_size,cl_uint num_events_in_wait_list,const cl_event *event_wait_list,cl_event *event);`\
>The number of dimensions used to specify the global work-items and work-items in the work-group. work_dim must be greater than zero and less than or equal to three.

13. Lecture du buffer de sortie avec clEnqueueReadBuffer\
`ret = clEnqueueReadBuffer(command_queue, c_mem_obj, CL_TRUE, 0, LIST_SIZE * sizeof(int), C, 0, NULL, NULL);`

14. Liberation des differents elements
```ret = clFlush(command_queue);
ret = clFinish(command_queue);
ret = clReleaseKernel(kernel);
ret = clReleaseProgram(program);
ret = clReleaseMemObject(a_mem_obj);
ret = clReleaseCommandQueue(command_queue);
ret = clReleaseContext(context);```
