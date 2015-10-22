set(CSD ${CSD}/vendor)
set(vendor_src
	${CSD}/avt.c
	${CSD}/avt.h
	${CSD}/pixelink.c
	${CSD}/pixelink.h
	${CSD}/basler.c
	${CSD}/basler_sff_registry.h
	${CSD}/basler_sff_registry.c
	PARENT_SCOPE)

install(FILES
	${CSD}/avt.h
	${CSD}/pixelink.h
	${CSD}/basler.h
	${CSD}/basler_sff.h
	DESTINATION include/dc1394/vendor)



