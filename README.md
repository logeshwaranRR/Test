@Repository
public interface ApplicantRepository extends JpaRepository<Applicant, Long> {

    @Query(value = """
        SELECT 
            a.first_name AS firstName,
            a.last_name AS lastName,
            a.birth_date AS birthDate,
            pn.phone_number AS phoneNumber,
            pt.type_name AS phoneType
        FROM iom_order o
        JOIN applicant_order_link aol ON o.order_id = aol.order_id
        JOIN applicant a ON aol.applicant_id = a.applicant_id
        LEFT JOIN phone_number pn ON a.applicant_id = pn.applicant_id
        LEFT JOIN phone_type pt ON pn.phone_type_id = pt.phone_type_id
        WHERE o.order_id = :orderId
        """, nativeQuery = true)
    List<ApplicantPhoneDTO> findApplicantAndPhoneByOrderId(@Param("orderId") Long orderId);
}
