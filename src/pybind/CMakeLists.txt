set(PYMODULE_NAME "neuroframe")
# Copied from https://github.com/pybind/cmake_example
pybind11_add_module(${PYMODULE_NAME} MODULE
	basic.cc
	cgraph.cc
	misc.cc
	ops.cc
	optim.cc
	pybind.cc
	tensor.cc
)

target_link_libraries(${PYMODULE_NAME} PRIVATE
	neuroframe_backend_cpu
	neuroframe_backend_cuda
	neuroframe_basic
	neuroframe_cgraph
	neuroframe_op
	neuroframe_optim
	neuroframe_tensor
)

target_compile_options(${PYMODULE_NAME} PRIVATE
	-flto=auto	# Supress the warning "lto-wrapper: warning: using serial compilation of 2 LTRANS jobs"
)

# Use stubgen to create .pyi files to sit alongside the just-built python module
# Copied from https://stackoverflow.com/a/77456943/16569836
find_program(Python_Executable python)
if(Python_Executable)
    set(Stubgen_Executable ${Python_Executable})
    set(Stubgen_Args -m mypy.stubgen)
    message(STATUS "Using mypy.stubgen for type stub generation")
else()
    find_program(Stubgen_Executable stubgen)
    if(Stubgen_Executable)
        set(Stubgen_Args "")
        message(STATUS "Using system stubgen for type stub generation")
    else()
        message(WARNING "No stubgen tool found, skipping type stub generation")
        set(Stubgen_Executable "")
    endif()
endif()

if(Stubgen_Executable)
    add_custom_command(TARGET ${PYMODULE_NAME} POST_BUILD
        COMMAND ${Stubgen_Executable} ${Stubgen_Args} ${PYMODULE_NAME} -o .
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}
        COMMENT "Generate .pyi files for IDE support"
    )
endif()
