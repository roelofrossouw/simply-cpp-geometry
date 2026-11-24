# What every test in this directory links against (sc-core for sc_test.h and the console).
set(SC_TEST_LINK_LIBRARIES sc-geometry sc::sc-core)

add_sc_test(point LABELS unit)
add_sc_test(rect LABELS unit)
add_sc_test(geometry LABELS unit)
add_sc_test(path LABELS unit)
add_sc_test(transform LABELS unit)
add_sc_test(dual LABELS unit)
add_sc_test(percent LABELS unit)
add_sc_test(distance LABELS unit)
add_sc_test(matrix LABELS unit)
